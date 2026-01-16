# How Loom Works

A comprehensive guide to understanding Loom's architecture, concepts, and operation.

---

## Executive Summary

**Loom is an AI-powered coding agent built entirely in Rust.** It provides a conversational interface for developers to interact with LLMs (Large Language Models) that can execute tools to perform file operations, code analysis, shell commands, and more.

### Key Value Propositions

| Feature | Description |
|---------|-------------|
| **Conversation-First** | All interactions are organized as "threads" (persistent conversations) |
| **Secure by Design** | API keys never leave the server; clients use a proxy |
| **Tool-Augmented AI** | LLMs can read files, edit code, run commands, search the web |
| **Multi-Provider** | Supports Anthropic Claude, OpenAI GPT, Google Vertex AI |
| **Streaming UX** | Real-time response display via Server-Sent Events |
| **Remote Execution** | "Weavers" provide isolated K8s pods for safe code execution |
| **Observable** | Structured logging, tracing, and audit trails throughout |

### Architecture at a Glance

```
┌─────────────────────────────────────────────────────────────────────┐
│                           USER INTERFACES                            │
├─────────────────┬──────────────────────┬────────────────────────────┤
│   loom-cli      │     loom-web         │     API Clients            │
│   (Terminal)    │     (Browser)        │     (Programmatic)         │
└────────┬────────┴──────────┬───────────┴────────────┬───────────────┘
         │                   │                        │
         │                   ▼                        │
         │         ┌─────────────────┐                │
         └────────▶│   loom-server   │◀───────────────┘
                   │  (HTTP + SSE)   │
                   └───────┬─────────┘
                           │
         ┌─────────────────┼─────────────────┐
         ▼                 ▼                 ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  LLM Providers  │ │    Database     │ │    Weavers      │
│ Claude/GPT/etc  │ │    (SQLite)     │ │  (K8s Pods)     │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

---

## Full Details

### 1. The Agent System (`loom-common-core`)

The heart of Loom is an **event-driven state machine** that orchestrates conversations between users, LLMs, and tools.

#### State Machine States

```
                    ┌──────────────────────┐
                    │  WaitingForUserInput │ ◀──────────────────┐
                    └──────────┬───────────┘                    │
                               │ (user sends message)           │
                               ▼                                │
                    ┌──────────────────────┐                    │
            ┌──────▶│     CallingLlm       │◀─────────┐         │
            │       └──────────┬───────────┘          │         │
            │                  │ (response received)  │         │
            │                  ▼                      │         │
            │       ┌──────────────────────┐          │         │
            │       │ProcessingLlmResponse │          │         │
            │       └──────────┬───────────┘          │         │
            │                  │                      │         │
            │    ┌─────────────┼─────────────┐        │         │
            │    │ (has tools) │ (no tools)  │        │         │
            │    ▼             │             ▼        │         │
            │ ┌────────────────┴──┐   ┌─────────────┐ │         │
            │ │  ExecutingTools   │   │   Done      │─┼─────────┘
            │ └────────┬──────────┘   └─────────────┘ │
            │          │ (tools complete)             │
            │          ▼                              │
            │ ┌──────────────────────┐                │
            │ │   PostToolsHook      │────────────────┘
            │ └──────────────────────┘  (loop for follow-up)
            │
            └── (retry on error)
```

#### Key Design Principle: Pure State Machine

The agent is **purely synchronous and deterministic**. It doesn't perform I/O directly. Instead, it returns `AgentAction` values that tell the caller what to do:

```rust
enum AgentAction {
    WaitForInput,                    // Ready for next message
    SendLlmRequest(LlmRequest),      // Make LLM API call
    ExecuteTools(Vec<ToolCall>),     // Run tool invocations
    RunPostToolsHook,                // Execute post-tool operations
    DisplayText(String),             // Show text to user
}
```

This separation makes the core logic:
- **Testable**: No mocking needed, just feed events and check outputs
- **Reusable**: Same state machine for CLI, web, and API
- **Debuggable**: All state transitions are explicit and logged

### 2. LLM Integration

#### Server-Side Proxy Architecture

Loom uses a **server-side proxy** for all LLM calls. The client never sees API keys.

```
┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
│    loom-cli     │   HTTP   │   loom-server   │   API    │    Anthropic    │
│                 │─────────▶│                 │─────────▶│     OpenAI      │
│ ProxyLlmClient  │◀─ SSE ───│   LlmService    │◀─ SSE ───│    Vertex AI    │
└─────────────────┘          └─────────────────┘          └─────────────────┘
```

**Benefits:**
- API keys stay on the server (security)
- Easy key rotation without client updates
- Centralized audit logging
- Usage tracking and rate limiting
- No secrets in distributed binaries

#### Supported Providers

| Provider | Crate | Models |
|----------|-------|--------|
| Anthropic | `loom-server-llm-anthropic` | Claude 3.5 Sonnet, Claude 3 Opus, etc. |
| OpenAI | `loom-server-llm-openai` | GPT-4o, GPT-4 Turbo, etc. |
| Google | `loom-server-llm-vertex` | Gemini Pro, etc. |

#### Streaming Protocol

Responses stream via **Server-Sent Events (SSE)**:

```
event: text_delta
data: {"content": "Let me "}

event: text_delta
data: {"content": "help you with that."}

event: tool_call_delta
data: {"call_id": "tc_123", "tool_name": "read_file", "arguments": "{\"path\":"}

event: tool_call_delta
data: {"call_id": "tc_123", "arguments": "\"/src/main.rs\"}"}

event: completed
data: {"full_response": {...}}
```

### 3. Tool System (`loom-cli-tools`)

Tools extend the agent's capabilities beyond text generation.

#### Tool Interface

```rust
#[async_trait]
pub trait Tool: Send + Sync {
    fn name(&self) -> &str;
    fn description(&self) -> &str;
    fn input_schema(&self) -> serde_json::Value;  // JSON Schema

    async fn invoke(
        &self,
        args: serde_json::Value,
        ctx: &ToolContext
    ) -> Result<serde_json::Value, ToolError>;
}
```

#### Built-in Tools

| Tool | Purpose | Example Use |
|------|---------|-------------|
| `read_file` | Read file contents | Examine source code |
| `list_files` | List directory | Explore project structure |
| `edit_file` | Modify files | Refactor code |
| `bash` | Run shell commands | Build, test, git operations |
| `web_search` | Search the internet | Research APIs, docs |
| `oracle` | Query secondary LLM | Get alternative perspectives |

#### Security Measures

- **Path validation**: Canonicalization prevents traversal attacks
- **Workspace boundaries**: Tools can't access files outside the workspace
- **Timeouts**: Commands have configurable time limits (default 60s)
- **Output limits**: Large outputs are truncated (default 256KB)

### 4. Thread Persistence (`loom-common-thread`)

Conversations are first-class citizens in Loom.

#### Thread Structure

```rust
pub struct Thread {
    pub id: ThreadId,              // "T-" prefixed UUID7
    pub version: u64,              // Optimistic concurrency control
    pub created_at: String,        // RFC3339 timestamp
    pub workspace_root: Option<String>,
    pub provider: Option<String>,  // "anthropic", "openai", etc.
    pub model: Option<String>,
    pub title: Option<String>,
    pub tags: Vec<String>,
    pub visibility: ThreadVisibility,
    pub conversation: ConversationSnapshot,
    pub agent_state: AgentStateSnapshot,
    pub metadata: ThreadMetadata,
}
```

#### Storage Locations

| Location | Path | Purpose |
|----------|------|---------|
| Local | `$XDG_DATA_HOME/loom/threads/<id>.json` | Fast access |
| Remote | SQLite on server | Sync, search, sharing |
| Pending | `$XDG_STATE_HOME/loom/sync/pending.json` | Offline queue |

#### Sync Behavior

1. **Auto-sync**: Threads sync after each agent turn completes
2. **Offline support**: Changes queue locally when offline
3. **Privacy**: Threads marked private NEVER sync
4. **Conflict resolution**: Optimistic concurrency with version numbers

### 5. Server Architecture (`loom-server`)

The server is built with Axum and provides 80+ HTTP endpoints.

#### Route Organization

```rust
// Public routes (no auth required)
PublicRouter::new()
    .route("/health", get(health))
    .route("/bin/:platform", get(download_binary))

// Authenticated routes
AuthedRouter::new()
    .route("/v1/threads", get(list_threads).post(create_thread))
    .route("/v1/threads/:id", get(get_thread).put(update_thread))
    .route("/proxy/:provider/stream", post(stream_completion))

// Optional auth (different behavior if logged in)
OptionalAuthRouter::new()
    .route("/v1/threads/:id/public", get(get_public_thread))
```

#### Key Server Features

- **LLM Proxy**: Secure credential handling
- **Thread API**: CRUD + search + sharing
- **Weaver Management**: K8s pod provisioning
- **Authentication**: OAuth + magic links
- **Analytics**: Event tracking, feature flags
- **Binary Distribution**: Self-update support

### 6. Weavers (Remote Execution)

Weavers are **ephemeral Kubernetes pods** for isolated code execution.

#### Why Weavers?

- **Isolation**: User code runs in sandboxed containers
- **Reproducibility**: Consistent environments
- **Security**: No access to production systems
- **Scalability**: Spin up/down on demand

#### Weaver Lifecycle

```
1. User requests weaver:     loom new --image python:3.12
2. Server provisions pod:    Creates K8s pod spec
3. K8s schedules pod:        Finds node, pulls image
4. Weaver initializes:       Sets up workspace, network
5. User attaches:            loom attach <weaver-id>
6. Agent executes in pod:    Tools run in container
7. TTL expires or delete:    Pod cleaned up (default 4h)
```

#### Weaver Features

- Custom images (Python, Node, Rust, etc.)
- TTL management (auto-cleanup)
- WireGuard tunnels for SSH access
- eBPF syscall auditing
- Prometheus metrics

### 7. Web Frontend (`loom-web`)

A modern Svelte 5 + SvelteKit application.

#### Tech Stack

| Technology | Purpose |
|------------|---------|
| Svelte 5 | Reactive UI framework |
| SvelteKit | Full-stack meta-framework |
| Tailwind CSS | Utility-first styling |
| XState | Complex state machines |
| LinguiJS | Internationalization |
| TypeScript | Type safety |

#### Key UI Components

- **Thread List**: Browse, search, filter conversations
- **Thread View**: Real-time conversation display
- **Agent State Badge**: Visual state indicator
- **Tool Status Badge**: Tool execution feedback
- **Theme Provider**: Light/dark mode support

### 8. Authentication & Authorization

#### Supported Auth Methods

| Method | Use Case |
|--------|----------|
| GitHub OAuth | Developer authentication |
| Google OAuth | Enterprise SSO |
| Okta OAuth | Enterprise SSO |
| Magic Links | Passwordless email auth |

#### ABAC (Attribute-Based Access Control)

Fine-grained permissions based on:
- User attributes (role, organization)
- Resource attributes (visibility, ownership)
- Environment attributes (time, location)

---

## Terminology Explanation

### Core Concepts

| Term | Definition |
|------|------------|
| **Thread** | A persistent conversation session with an LLM agent |
| **Agent** | The state machine that orchestrates conversations |
| **Tool** | A capability the agent can invoke (read files, run commands, etc.) |
| **Weaver** | An ephemeral K8s pod for remote code execution |
| **Provider** | An LLM service (Anthropic, OpenAI, Vertex AI) |

### Technical Terms

| Term | Definition |
|------|------------|
| **SSE** | Server-Sent Events; one-way streaming from server to client |
| **ABAC** | Attribute-Based Access Control; fine-grained permissions |
| **Proxy LLM Client** | Client that talks to server instead of LLM directly |
| **Thread Sync** | Bidirectional synchronization of threads between local/remote |
| **Post-Tools Hook** | Operations run after tools complete (e.g., auto-commit) |

### State Machine States

| State | Description |
|-------|-------------|
| **WaitingForUserInput** | Idle, ready for user message |
| **CallingLlm** | LLM request in flight |
| **ProcessingLlmResponse** | Examining LLM response |
| **ExecutingTools** | Running tool invocations |
| **PostToolsHook** | Running follow-up operations |
| **Error** | Recoverable error occurred |
| **ShuttingDown** | Graceful shutdown |

---

## Step-by-Step Guidance

### Getting Started

#### 1. Install Loom CLI

```bash
# Download for your platform
curl -fsSL https://loom.ghuntley.com/bin/linux-x86_64 -o loom
chmod +x loom
sudo mv loom /usr/local/bin/

# Or on macOS
curl -fsSL https://loom.ghuntley.com/bin/darwin-aarch64 -o loom
chmod +x loom
sudo mv loom /usr/local/bin/
```

#### 2. Authenticate

```bash
# Login with your account
loom --server-url https://loom.ghuntley.com login
```

#### 3. Start a Conversation

```bash
# Start a new REPL session
loom

# Or start in a specific directory
cd /path/to/project && loom
```

#### 4. Interact with the Agent

```
You: Read the README.md file and summarize what this project does.

Agent: [Reads file, provides summary]

You: Add a new function to src/lib.rs that calculates fibonacci numbers.

Agent: [Reads file, uses edit_file tool, shows changes]
```

### Common Operations

#### Resume Previous Conversation

```bash
# List recent threads
loom list

# Resume a specific thread
loom resume T-abc123def456
```

#### Search Conversations

```bash
# Search across all threads
loom search "fibonacci"
```

#### Use Private Mode

```bash
# Start a session that never syncs to server
loom private
```

#### Share a Thread

```bash
# Make a thread visible to your organization
loom share T-abc123 --visibility organization

# Make it public
loom share T-abc123 --visibility public
```

### Development Workflow

#### Building the Project

```bash
# Using Cargo (development)
cargo build --workspace

# Using Nix (reproducible)
nix build .#loom-cli-c2n
```

#### Running Tests

```bash
# All tests
cargo test --workspace

# Specific crate
cargo test -p loom-common-core

# Specific test
cargo test -p loom-common-core test_agent_state_transitions
```

#### Running the Server Locally

```bash
# Build and run
cargo build --release
LOOM_SERVER_PORT=9090 \
LOOM_SERVER_DB_PATH=/tmp/loom.db \
LOOM_SERVER_AUTH_DEV_MODE=1 \
./target/release/loom-server
```

---

## Ideas for Products and Platforms

### 1. Enterprise Development Assistant

**Platform**: Internal development tool for organizations

**Features**:
- Integrate with corporate Git repos (GitHub Enterprise, GitLab)
- SCIM provisioning for user management
- Custom tool development for internal APIs
- Audit logging for compliance
- Fine-grained access control per project

**Value**: Accelerate development while maintaining security and compliance.

### 2. Educational Coding Platform

**Platform**: Interactive learning environment

**Features**:
- Pre-configured weavers for different languages/frameworks
- Guided tutorials with AI assistance
- Progress tracking and assessment
- Collaborative sessions with instructors
- Safe sandbox execution for student code

**Value**: Learn programming with AI pair programming.

### 3. Code Review Service

**Platform**: Automated code review tool

**Features**:
- Integration with PR workflows
- Automated review comments
- Security vulnerability detection
- Style/convention enforcement
- Metrics and reporting

**Value**: Faster, more thorough code reviews.

### 4. DevOps Automation Platform

**Platform**: Infrastructure management assistant

**Features**:
- Read/write Terraform, Kubernetes configs
- Execute infrastructure commands safely
- Rollback and recovery assistance
- Cost optimization suggestions
- Incident response automation

**Value**: AI-assisted infrastructure management.

### 5. Technical Documentation Generator

**Platform**: Documentation automation

**Features**:
- Analyze codebase structure
- Generate API documentation
- Create architecture diagrams
- Update docs with code changes
- Multi-language support

**Value**: Always up-to-date documentation.

### 6. Bug Triage and Resolution System

**Platform**: Issue management integration

**Features**:
- Connect to Jira, Linear, GitHub Issues
- Analyze bug reports with codebase context
- Suggest fixes with confidence scores
- Generate test cases for regressions
- Priority recommendations

**Value**: Faster bug resolution.

### 7. Multi-Agent Development Team

**Platform**: Coordinated AI agents

**Features**:
- Architect agent for design decisions
- Developer agents for implementation
- Reviewer agent for quality
- Tester agent for verification
- Orchestrator for coordination

**Value**: Scalable AI development assistance.

---

## Ideas for Improvements

### Near-Term Enhancements

#### 1. Improved Context Management
- **Sliding window context**: Automatically summarize old messages
- **Semantic chunking**: Keep relevant context, drop redundant
- **Context injection**: Automatically include relevant files

#### 2. Tool Enhancements
- **Git-aware editing**: Auto-stage changes, suggest commits
- **Test integration**: Run tests after edits, report results
- **Type checking**: Integrate with LSP for real-time feedback

#### 3. Better Collaboration
- **Real-time collaboration**: Multiple users in same thread
- **Thread branching**: Fork conversations to explore alternatives
- **Thread templates**: Reusable conversation starters

#### 4. Enhanced Search
- **Semantic search**: Find threads by meaning, not just keywords
- **Cross-thread references**: Link related conversations
- **Conversation analytics**: Insights on usage patterns

### Medium-Term Improvements

#### 5. Agent Specialization
- **Role-based agents**: Frontend, backend, DevOps specialists
- **Custom personas**: Adjustable communication styles
- **Domain expertise**: Deep knowledge of specific frameworks

#### 6. Workflow Automation
- **Scheduled tasks**: Cron-like agent execution
- **Webhooks**: Trigger agents from external events
- **Pipeline integration**: CI/CD integration

#### 7. Enhanced Safety
- **Sandboxed tool execution**: Run tools in isolated containers
- **Change previews**: Show diffs before applying
- **Undo support**: Revert tool effects

### Long-Term Vision

#### 8. Multi-Modal Support
- **Image understanding**: Analyze screenshots, diagrams
- **Voice interface**: Speak with the agent
- **IDE integration**: Inline agent in editors

#### 9. Learning and Adaptation
- **User preferences**: Learn coding style from history
- **Project conventions**: Adapt to project-specific patterns
- **Feedback loops**: Improve from explicit feedback

#### 10. Distributed Agents
- **Agent coordination**: Multiple agents working together
- **Task distribution**: Parallelize work across agents
- **Consensus mechanisms**: Agreement on solutions

---

## Factorio-Style Frontend Add-On

### Concept: The Development Factory

Imagine a **visual, factory-building interface** for software development, inspired by Factorio. Instead of placing belts and machines to produce items, you place **pipelines and processing nodes** to produce code.

### Core Metaphor

| Factorio Concept | Development Factory Concept |
|------------------|----------------------------|
| Raw materials | Requirements, specs, issues |
| Machines | Agent nodes (Architect, Developer, Reviewer) |
| Belts | Data flow (code, reviews, tests) |
| Inserters | Triggers and connectors |
| Products | Commits, PRs, releases |
| Power | API tokens, compute budget |
| Research | Model fine-tuning, tool development |

### Visual Layout

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        THE DEVELOPMENT FACTORY                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────┐     │
│   │ ISSUES  │───▶│  ARCHITECT  │───▶│  DEVELOPER  │───▶│ TESTS   │     │
│   │  INBOX  │    │    AGENT    │    │    AGENT    │    │ RUNNER  │     │
│   └─────────┘    └──────┬──────┘    └──────┬──────┘    └────┬────┘     │
│                         │                  │                 │          │
│                         │     ┌────────────▼──────────┐     │          │
│                         │     │    CODE REPOSITORY    │     │          │
│                         │     │     (Git Storage)     │     │          │
│                         │     └────────────┬──────────┘     │          │
│                         │                  │                 │          │
│                         │                  ▼                 │          │
│                         │    ┌─────────────────────────┐    │          │
│                         └───▶│      REVIEW AGENT       │◀───┘          │
│                              │  (Quality Assurance)    │               │
│                              └───────────┬─────────────┘               │
│                                          │                              │
│                                          ▼                              │
│                              ┌─────────────────────────┐               │
│                              │     PR ASSEMBLY LINE    │               │
│                              │  (Merge & Deploy)       │               │
│                              └─────────────────────────┘               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Key Components

#### 1. Node Types (Machines)

| Node | Icon | Function |
|------|------|----------|
| **Inbox** | 📥 | Receives issues, specs, requirements |
| **Architect Agent** | 🏗️ | Designs solutions, creates plans |
| **Developer Agent** | 💻 | Writes and edits code |
| **Reviewer Agent** | 🔍 | Reviews code, suggests improvements |
| **Test Runner** | 🧪 | Executes test suites |
| **Deployer** | 🚀 | Handles releases and deployments |
| **Splitter** | ⑂ | Divides work into parallel tracks |
| **Merger** | ⊕ | Combines results from parallel work |

#### 2. Conveyor Types (Data Flow)

| Belt | Color | Contents |
|------|-------|----------|
| **Issue Belt** | Yellow | Issues, bugs, feature requests |
| **Code Belt** | Blue | Source files, patches, diffs |
| **Review Belt** | Purple | Review comments, approvals |
| **Test Belt** | Green | Test results, coverage reports |
| **Artifact Belt** | Orange | Built artifacts, binaries |

#### 3. Signals and Control

| Signal | Meaning |
|--------|---------|
| 🟢 Green | Process healthy, items flowing |
| 🟡 Yellow | Bottleneck, queue building up |
| 🔴 Red | Blocked, needs attention |
| ⚡ Lightning | High throughput mode |

### User Interactions

#### Placing and Connecting Nodes

```
1. Drag "Developer Agent" from toolbar
2. Drop on canvas
3. Configure agent (model, tools, context)
4. Connect input belt (from Issue Inbox)
5. Connect output belt (to Code Repository)
6. Agent starts processing when items arrive
```

#### Monitoring the Factory

```
┌──────────────────────────────────────────────────┐
│             FACTORY DASHBOARD                     │
├──────────────────────────────────────────────────┤
│ Throughput:     ████████░░ 8 items/hour          │
│ Quality:        █████████░ 92% pass rate         │
│ Token Usage:    ███░░░░░░░ 30% of budget         │
│ Active Agents:  5/10                             │
│ Queue Depth:    12 items waiting                 │
├──────────────────────────────────────────────────┤
│ Recent Activity:                                  │
│ • Architect designed PR #234         2m ago      │
│ • Developer implemented login fix    5m ago      │
│ • Tests passed for feature-x         8m ago      │
│ • Reviewer approved PR #232         15m ago      │
└──────────────────────────────────────────────────┘
```

#### Blueprint System

Save and share factory configurations:

```yaml
name: "Standard Feature Pipeline"
description: "Issue → Design → Code → Test → Review → Merge"
nodes:
  - type: inbox
    id: issues
    source: github-issues
  - type: architect_agent
    id: architect
    model: claude-3-5-sonnet
    input: issues.output
  - type: developer_agent
    id: developer
    model: claude-3-5-sonnet
    input: architect.plan
    tools: [read_file, edit_file, bash]
  - type: test_runner
    id: tests
    input: developer.changes
    on_fail: developer.retry
  - type: reviewer_agent
    id: reviewer
    input: tests.passed
    model: claude-3-opus
belts:
  - from: issues → architect
  - from: architect → developer
  - from: developer → tests
  - from: tests → reviewer
```

### Advanced Features

#### 1. Logistics Network (Remote Execution)

Like Factorio's logistics robots, **Weavers** are autonomous workers:

```
┌───────────────────────────────────────────────┐
│         LOGISTICS NETWORK                      │
├───────────────────────────────────────────────┤
│ Weavers Available: 5                          │
│ Weavers Active:    3                          │
│ Weavers Charging:  2                          │
│                                                │
│ Active Tasks:                                  │
│ 🤖 weaver-01: Building frontend components    │
│ 🤖 weaver-02: Running integration tests       │
│ 🤖 weaver-03: Generating API documentation    │
│                                                │
│ [Request New Weaver]  [View Weaver Network]   │
└───────────────────────────────────────────────┘
```

#### 2. Research Tree (Model & Tool Upgrades)

```
                    ┌───────────────────┐
                    │   Basic Tools     │
                    │   (read, write)   │
                    └─────────┬─────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
     ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
     │ Git Tools   │  │ Web Search  │  │ Database    │
     │ (commit,    │  │ (research)  │  │ Tools       │
     │  branch)    │  │             │  │             │
     └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
            │                │                │
            ▼                ▼                ▼
     ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
     │ CI/CD       │  │ API Client  │  │ Analytics   │
     │ Integration │  │ Generator   │  │ Tools       │
     └─────────────┘  └─────────────┘  └─────────────┘
```

#### 3. Achievements & Statistics

```
┌─────────────────────────────────────────────────┐
│           FACTORY ACHIEVEMENTS                   │
├─────────────────────────────────────────────────┤
│ 🏆 First Commit         - Ship your first code  │
│ 🏆 Assembly Line        - 10 PRs merged         │
│ 🏆 Quality Control      - 100% test pass streak │
│ 🏆 Speedrunner          - Issue to merge < 1hr  │
│ 🏆 Mass Production      - 100 items processed   │
│ 🏆 Zero Defects         - No bugs in a week     │
│ 🏆 The Factory Grows    - 10 agents running     │
├─────────────────────────────────────────────────┤
│ Lifetime Statistics:                             │
│ • Total items processed:        1,247           │
│ • Code lines written:          45,892           │
│ • Tests generated:              3,456           │
│ • Reviews completed:              892           │
│ • Time saved (estimated):       340 hours       │
└─────────────────────────────────────────────────┘
```

### Implementation Approach

#### Phase 1: Core Canvas
- Drag-and-drop node placement
- Belt connections with validation
- Basic node types (Agent, Test, Deploy)
- Real-time status updates

#### Phase 2: Agent Integration
- Connect to Loom backend
- Thread management per node
- Tool configuration UI
- SSE streaming for live updates

#### Phase 3: Workflow Automation
- Blueprint save/load
- Trigger conditions
- Parallel processing
- Error handling and retry

#### Phase 4: Advanced Features
- Weaver management
- Research/upgrade system
- Statistics and achievements
- Multi-user collaboration

### Technical Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     FACTORIO FRONTEND                            │
├─────────────────────────────────────────────────────────────────┤
│  Svelte 5 + SvelteKit                                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │  Canvas Layer   │  │  State Manager  │  │  API Client     │ │
│  │  (Pixi.js or    │  │  (XState)       │  │  (Loom API)     │ │
│  │   SVG/Canvas)   │  │                 │  │                 │ │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘ │
│           │                    │                    │           │
│           └────────────────────┼────────────────────┘           │
│                                ▼                                │
│                    ┌─────────────────────┐                      │
│                    │   Factory Store     │                      │
│                    │   (Nodes, Belts,    │                      │
│                    │    Status, Metrics) │                      │
│                    └─────────────────────┘                      │
├─────────────────────────────────────────────────────────────────┤
│                     LOOM BACKEND                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │  Thread API     │  │  Agent Runtime  │  │  Weaver API     │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Why This Metaphor Works

1. **Visual Intuition**: Factory building games train spatial reasoning about workflows
2. **Optimization Mindset**: Players naturally want to improve throughput and efficiency
3. **Scalable Complexity**: Start simple, add sophistication gradually
4. **Satisfying Feedback**: Watching items flow through is inherently rewarding
5. **Collaborative Potential**: Multiple people can work on different factory sections
6. **Debugging Clarity**: Easy to see where things get stuck or break

---

## Conclusion

Loom represents a sophisticated approach to AI-assisted software development. Its clean architecture—separating the pure state machine from I/O operations, using a server-side proxy for security, and providing extensible tool and provider systems—makes it both powerful and maintainable.

The potential applications extend far beyond a simple coding assistant. From enterprise development platforms to educational tools to the imaginative Factorio-style development factory, Loom's foundation supports diverse and innovative product directions.

Whether you're using Loom as a daily development tool, building products on top of it, or contributing to its evolution, understanding these underlying mechanics will help you get the most out of this powerful system.
