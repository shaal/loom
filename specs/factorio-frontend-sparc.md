# SPARC: Factorio-Style Development Factory Frontend

> **SPARC**: Specification, Pseudocode, Architecture, Refinement, Completion

A comprehensive technical specification for implementing a Factorio-inspired visual workflow builder for Loom.

---

## Table of Contents

1. [Specification](#1-specification)
2. [Pseudocode](#2-pseudocode)
3. [Architecture](#3-architecture)
4. [Refinement](#4-refinement)
5. [Completion](#5-completion)

---

# 1. Specification

## 1.1 Project Overview

### 1.1.1 Vision Statement

Build a **visual, factory-building interface** for software development workflows where users create pipelines of AI agents, tools, and automation nodes connected by data flows—inspired by Factorio's satisfying logistics systems.

### 1.1.2 Core Metaphor Mapping

| Factorio Concept | Factory Frontend Concept | Implementation |
|------------------|--------------------------|----------------|
| Machines | Agent Nodes | Svelte components with Loom thread integration |
| Conveyor Belts | Data Flow Connections | SVG paths with animated particles |
| Items | Work Items | Issues, code changes, reviews, artifacts |
| Inserters | Triggers & Filters | Conditional routing logic |
| Power Grid | API Token Budget | Usage tracking and limits |
| Logistics Robots | Weavers | K8s pod workers |
| Blueprints | Saved Workflows | JSON/YAML configurations |
| Research | Upgrades | Model and tool unlocks |
| Pollution | Technical Debt | Code quality metrics |

### 1.1.3 Target Users

1. **Development Team Leads**: Design team workflows
2. **DevOps Engineers**: Build CI/CD automation
3. **Individual Developers**: Personal productivity pipelines
4. **Platform Engineers**: Create reusable workflow templates

## 1.2 Functional Requirements

### 1.2.1 Canvas System

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-C01 | Infinite pannable canvas with zoom (0.1x - 4x) | P0 |
| FR-C02 | Grid-based node snapping (32px grid) | P1 |
| FR-C03 | Multi-select with box selection | P1 |
| FR-C04 | Copy/paste nodes and connections | P1 |
| FR-C05 | Undo/redo with 100-step history | P0 |
| FR-C06 | Minimap for large factories | P2 |
| FR-C07 | Keyboard shortcuts (delete, duplicate, etc.) | P1 |

### 1.2.2 Node System

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-N01 | Drag-and-drop node placement from palette | P0 |
| FR-N02 | Node configuration panel on selection | P0 |
| FR-N03 | Visual state indicators (idle, working, error) | P0 |
| FR-N04 | Input/output port system with type safety | P0 |
| FR-N05 | Node grouping into sub-factories | P2 |
| FR-N06 | Custom node creation (advanced) | P3 |

### 1.2.3 Connection System

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-X01 | Drag-to-connect ports with visual feedback | P0 |
| FR-X02 | Connection validation (type compatibility) | P0 |
| FR-X03 | Animated item flow on belts | P1 |
| FR-X04 | Connection labels for clarity | P2 |
| FR-X05 | Splitter nodes for fan-out | P1 |
| FR-X06 | Merger nodes for fan-in | P1 |

### 1.2.4 Agent Integration

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-A01 | Create Loom thread per agent node | P0 |
| FR-A02 | Stream agent responses in real-time | P0 |
| FR-A03 | Configure model, tools, system prompt | P0 |
| FR-A04 | View agent conversation history | P1 |
| FR-A05 | Manual intervention capability | P1 |
| FR-A06 | Agent pause/resume controls | P1 |

### 1.2.5 Workflow Execution

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-W01 | Start/stop factory execution | P0 |
| FR-W02 | Real-time status updates via WebSocket | P0 |
| FR-W03 | Queue management for pending items | P1 |
| FR-W04 | Error handling with retry options | P0 |
| FR-W05 | Execution history and logs | P1 |
| FR-W06 | Scheduled/triggered execution | P2 |

### 1.2.6 Blueprint System

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-B01 | Save factory as blueprint | P0 |
| FR-B02 | Load blueprint to canvas | P0 |
| FR-B03 | Blueprint library (personal/shared) | P1 |
| FR-B04 | Blueprint versioning | P2 |
| FR-B05 | Blueprint import/export (JSON/YAML) | P1 |
| FR-B06 | Blueprint marketplace | P3 |

## 1.3 Non-Functional Requirements

### 1.3.1 Performance

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-P01 | Canvas render at 60fps with 100 nodes | Required |
| NFR-P02 | Node placement latency < 16ms | Required |
| NFR-P03 | Connection drawing latency < 16ms | Required |
| NFR-P04 | Initial load time < 2s | Target |
| NFR-P05 | Memory usage < 200MB for typical factory | Target |

### 1.3.2 Scalability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-S01 | Support 500+ nodes per factory | Required |
| NFR-S02 | Support 1000+ connections | Required |
| NFR-S03 | Support 50 concurrent agent executions | Target |

### 1.3.3 Reliability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-R01 | Auto-save every 30 seconds | Required |
| NFR-R02 | Crash recovery from auto-save | Required |
| NFR-R03 | Graceful degradation on network loss | Required |

### 1.3.4 Accessibility

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-A01 | Keyboard navigation for all actions | Required |
| NFR-A02 | Screen reader support for node info | Target |
| NFR-A03 | High contrast mode | Target |
| NFR-A04 | Reduced motion mode | Required |

## 1.4 Node Type Specifications

### 1.4.1 Source Nodes (Inputs)

```yaml
IssueInbox:
  description: "Receives issues from external sources"
  inputs: []
  outputs:
    - name: issues
      type: Issue[]
  config:
    - source: enum[github, jira, linear, manual]
    - filters: IssueFilter[]
    - polling_interval: duration

ManualInput:
  description: "User-provided input items"
  inputs: []
  outputs:
    - name: items
      type: WorkItem[]
  config:
    - item_type: enum[text, code, file, url]

WebhookReceiver:
  description: "Receives items via HTTP webhook"
  inputs: []
  outputs:
    - name: payload
      type: JSON
  config:
    - endpoint: string (auto-generated)
    - secret: string
    - transform: JSONPath

ScheduleTrigger:
  description: "Triggers on a schedule"
  inputs: []
  outputs:
    - name: trigger
      type: TriggerEvent
  config:
    - cron: string
    - timezone: string
```

### 1.4.2 Agent Nodes (Processing)

```yaml
ArchitectAgent:
  description: "Designs solutions and creates implementation plans"
  inputs:
    - name: requirements
      type: WorkItem
  outputs:
    - name: plan
      type: ImplementationPlan
    - name: rejected
      type: WorkItem  # Items that can't be processed
  config:
    - model: enum[claude-3-5-sonnet, claude-3-opus, gpt-4o]
    - system_prompt: string
    - context_files: string[]  # Globs for relevant files
    - max_tokens: number

DeveloperAgent:
  description: "Implements code changes based on plans"
  inputs:
    - name: plan
      type: ImplementationPlan
  outputs:
    - name: changes
      type: CodeChanges
    - name: failed
      type: ImplementationPlan
  config:
    - model: enum[claude-3-5-sonnet, gpt-4o]
    - tools: enum[read_file, edit_file, bash, list_files][]
    - workspace: string
    - auto_commit: boolean

ReviewerAgent:
  description: "Reviews code changes for quality"
  inputs:
    - name: changes
      type: CodeChanges
  outputs:
    - name: approved
      type: CodeChanges
    - name: needs_revision
      type: ReviewFeedback
  config:
    - model: enum[claude-3-opus, gpt-4o]
    - review_criteria: string[]
    - auto_approve_threshold: number (0-100)

CustomAgent:
  description: "User-defined agent with custom behavior"
  inputs:
    - name: input
      type: any
  outputs:
    - name: output
      type: any
  config:
    - model: enum[...]
    - system_prompt: string
    - tools: Tool[]
    - input_schema: JSONSchema
    - output_schema: JSONSchema
```

### 1.4.3 Tool Nodes (Operations)

```yaml
TestRunner:
  description: "Executes test suites"
  inputs:
    - name: changes
      type: CodeChanges
  outputs:
    - name: passed
      type: TestResult
    - name: failed
      type: TestResult
  config:
    - command: string
    - timeout: duration
    - coverage_threshold: number

CodeAnalyzer:
  description: "Static analysis and linting"
  inputs:
    - name: code
      type: CodeChanges
  outputs:
    - name: clean
      type: CodeChanges
    - name: issues
      type: AnalysisResult
  config:
    - tools: enum[eslint, clippy, mypy, etc.][]
    - severity_threshold: enum[error, warning, info]

GitOperations:
  description: "Git commands (commit, branch, push)"
  inputs:
    - name: changes
      type: CodeChanges
  outputs:
    - name: committed
      type: GitCommit
    - name: error
      type: GitError
  config:
    - operation: enum[commit, branch, push, pr]
    - branch_pattern: string
    - commit_message_template: string

BashExecutor:
  description: "Run arbitrary shell commands"
  inputs:
    - name: trigger
      type: any
  outputs:
    - name: success
      type: CommandResult
    - name: failure
      type: CommandResult
  config:
    - command: string
    - timeout: duration
    - working_directory: string
    - environment: Record<string, string>
```

### 1.4.4 Flow Control Nodes

```yaml
Splitter:
  description: "Splits items to multiple outputs"
  inputs:
    - name: input
      type: T
  outputs:
    - name: output_1
      type: T
    - name: output_2
      type: T
    # ... up to N outputs
  config:
    - mode: enum[round_robin, duplicate, conditional]
    - conditions: Condition[]  # For conditional mode

Merger:
  description: "Merges items from multiple inputs"
  inputs:
    - name: input_1
      type: T
    - name: input_2
      type: T
    # ... up to N inputs
  outputs:
    - name: output
      type: T
  config:
    - mode: enum[fifo, priority, wait_all]
    - priorities: number[]  # For priority mode

Filter:
  description: "Filters items based on conditions"
  inputs:
    - name: input
      type: T
  outputs:
    - name: matched
      type: T
    - name: unmatched
      type: T
  config:
    - condition: Expression

Buffer:
  description: "Queues items with capacity limit"
  inputs:
    - name: input
      type: T
  outputs:
    - name: output
      type: T
  config:
    - capacity: number
    - overflow_behavior: enum[drop_oldest, drop_newest, block]

RateLimiter:
  description: "Controls item flow rate"
  inputs:
    - name: input
      type: T
  outputs:
    - name: output
      type: T
  config:
    - rate: number
    - period: duration
    - burst: number
```

### 1.4.5 Sink Nodes (Outputs)

```yaml
PRCreator:
  description: "Creates GitHub/GitLab pull requests"
  inputs:
    - name: changes
      type: CodeChanges
  outputs: []
  config:
    - provider: enum[github, gitlab, bitbucket]
    - repository: string
    - base_branch: string
    - title_template: string
    - body_template: string
    - reviewers: string[]
    - labels: string[]

Notifier:
  description: "Sends notifications"
  inputs:
    - name: event
      type: any
  outputs: []
  config:
    - channel: enum[slack, email, discord, webhook]
    - template: string
    - recipients: string[]

ArtifactStore:
  description: "Stores build artifacts"
  inputs:
    - name: artifact
      type: Artifact
  outputs: []
  config:
    - storage: enum[s3, gcs, local]
    - path_template: string
    - retention: duration

Logger:
  description: "Logs items for debugging"
  inputs:
    - name: item
      type: any
  outputs:
    - name: passthrough
      type: any
  config:
    - format: enum[json, text, pretty]
    - include_timestamp: boolean
```

## 1.5 Data Type Specifications

### 1.5.1 Core Types

```typescript
// Base work item that flows through the factory
interface WorkItem {
  id: string;              // UUID
  type: WorkItemType;
  created_at: string;      // ISO 8601
  source_node: string;     // Node ID that created it
  payload: unknown;        // Type-specific data
  metadata: Record<string, unknown>;
  trace: TraceEntry[];     // Audit trail
}

type WorkItemType =
  | 'issue'
  | 'code_changes'
  | 'test_result'
  | 'review'
  | 'artifact'
  | 'notification'
  | 'custom';

interface TraceEntry {
  node_id: string;
  timestamp: string;
  action: string;
  duration_ms: number;
}

// Issue from external tracker
interface Issue extends WorkItem {
  type: 'issue';
  payload: {
    external_id: string;
    title: string;
    body: string;
    labels: string[];
    assignees: string[];
    priority: 'low' | 'medium' | 'high' | 'critical';
    url: string;
  };
}

// Implementation plan from architect
interface ImplementationPlan extends WorkItem {
  type: 'implementation_plan';
  payload: {
    summary: string;
    steps: PlanStep[];
    estimated_complexity: 'trivial' | 'simple' | 'moderate' | 'complex';
    affected_files: string[];
    risks: string[];
  };
}

interface PlanStep {
  order: number;
  description: string;
  file_path?: string;
  action: 'create' | 'modify' | 'delete';
}

// Code changes from developer
interface CodeChanges extends WorkItem {
  type: 'code_changes';
  payload: {
    branch: string;
    commit_sha?: string;
    files: FileChange[];
    summary: string;
  };
}

interface FileChange {
  path: string;
  action: 'create' | 'modify' | 'delete';
  diff?: string;
  content?: string;
}

// Test execution result
interface TestResult extends WorkItem {
  type: 'test_result';
  payload: {
    passed: boolean;
    total: number;
    passed_count: number;
    failed_count: number;
    skipped_count: number;
    coverage?: number;
    failures: TestFailure[];
    duration_ms: number;
  };
}

interface TestFailure {
  name: string;
  message: string;
  stack_trace?: string;
}

// Code review feedback
interface ReviewFeedback extends WorkItem {
  type: 'review';
  payload: {
    approved: boolean;
    score: number;  // 0-100
    comments: ReviewComment[];
    suggested_changes: SuggestedChange[];
  };
}

interface ReviewComment {
  file: string;
  line: number;
  severity: 'info' | 'warning' | 'error';
  message: string;
}

interface SuggestedChange {
  file: string;
  line_start: number;
  line_end: number;
  replacement: string;
  reason: string;
}
```

### 1.5.2 Factory Types

```typescript
// Complete factory definition
interface Factory {
  id: string;
  name: string;
  description: string;
  version: number;
  created_at: string;
  updated_at: string;
  owner_id: string;
  visibility: 'private' | 'organization' | 'public';

  // Canvas state
  canvas: CanvasState;

  // Node definitions
  nodes: FactoryNode[];

  // Connection definitions
  connections: Connection[];

  // Global configuration
  config: FactoryConfig;

  // Runtime state (not persisted in blueprint)
  runtime?: FactoryRuntime;
}

interface CanvasState {
  zoom: number;
  pan_x: number;
  pan_y: number;
  selected_nodes: string[];
  selected_connections: string[];
}

interface FactoryNode {
  id: string;
  type: NodeType;
  position: { x: number; y: number };
  size: { width: number; height: number };
  label: string;
  config: NodeConfig;

  // Runtime state
  status?: NodeStatus;
  queue_depth?: number;
  last_activity?: string;
  error?: string;
}

type NodeStatus =
  | 'idle'
  | 'working'
  | 'waiting'
  | 'error'
  | 'disabled';

interface Connection {
  id: string;
  source_node: string;
  source_port: string;
  target_node: string;
  target_port: string;

  // Visual properties
  color?: string;
  animated?: boolean;
  label?: string;
}

interface FactoryConfig {
  // Execution settings
  max_concurrent_agents: number;
  default_timeout: number;
  retry_policy: RetryPolicy;

  // Resource limits
  token_budget: number;
  weaver_limit: number;

  // Notifications
  notify_on_error: boolean;
  notify_on_complete: boolean;
  notification_channels: string[];
}

interface RetryPolicy {
  max_attempts: number;
  initial_delay_ms: number;
  max_delay_ms: number;
  backoff_multiplier: number;
}

interface FactoryRuntime {
  status: 'stopped' | 'running' | 'paused' | 'error';
  started_at?: string;
  items_processed: number;
  items_failed: number;
  active_agents: number;
  token_usage: number;
  errors: RuntimeError[];
}

interface RuntimeError {
  timestamp: string;
  node_id: string;
  message: string;
  stack_trace?: string;
  item_id?: string;
}
```

### 1.5.3 Blueprint Types

```typescript
// Shareable factory template
interface Blueprint {
  id: string;
  name: string;
  description: string;
  version: string;  // Semver
  author: string;
  tags: string[];
  created_at: string;
  updated_at: string;

  // Factory definition (without runtime state)
  factory: Omit<Factory, 'runtime' | 'owner_id'>;

  // Parameter definitions for customization
  parameters: BlueprintParameter[];

  // Usage statistics
  stats: BlueprintStats;
}

interface BlueprintParameter {
  name: string;
  description: string;
  type: 'string' | 'number' | 'boolean' | 'enum' | 'secret';
  default?: unknown;
  required: boolean;
  options?: string[];  // For enum type

  // Where this parameter is used
  bindings: ParameterBinding[];
}

interface ParameterBinding {
  node_id: string;
  config_path: string;  // JSONPath to config field
}

interface BlueprintStats {
  downloads: number;
  stars: number;
  forks: number;
  last_used: string;
}
```

---

# 2. Pseudocode

## 2.1 Canvas Rendering Engine

### 2.1.1 Main Render Loop

```pseudocode
CLASS CanvasRenderer
  PROPERTY canvas: HTMLCanvasElement
  PROPERTY ctx: CanvasRenderingContext2D
  PROPERTY zoom: number = 1.0
  PROPERTY pan: Vector2 = {x: 0, y: 0}
  PROPERTY nodes: Map<string, FactoryNode>
  PROPERTY connections: Map<string, Connection>
  PROPERTY selectedNodes: Set<string>
  PROPERTY animationFrame: number
  PROPERTY lastFrameTime: number

  METHOD start()
    this.lastFrameTime = performance.now()
    this.scheduleFrame()

  METHOD scheduleFrame()
    this.animationFrame = requestAnimationFrame(this.render.bind(this))

  METHOD render(currentTime: number)
    deltaTime = currentTime - this.lastFrameTime
    this.lastFrameTime = currentTime

    // Clear canvas
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)

    // Apply camera transform
    this.ctx.save()
    this.ctx.translate(this.pan.x, this.pan.y)
    this.ctx.scale(this.zoom, this.zoom)

    // Draw grid
    this.drawGrid()

    // Draw connections (below nodes)
    FOR connection IN this.connections.values()
      this.drawConnection(connection, deltaTime)
    END FOR

    // Draw nodes
    FOR node IN this.nodes.values()
      this.drawNode(node)
    END FOR

    // Draw selection box if dragging
    IF this.selectionBox != null
      this.drawSelectionBox()
    END IF

    // Draw connection preview if connecting
    IF this.connectionPreview != null
      this.drawConnectionPreview()
    END IF

    this.ctx.restore()

    // Draw UI overlay (not affected by camera)
    this.drawMinimap()
    this.drawToolbar()

    // Schedule next frame
    this.scheduleFrame()

  METHOD drawGrid()
    gridSize = 32
    startX = Math.floor(-this.pan.x / this.zoom / gridSize) * gridSize
    startY = Math.floor(-this.pan.y / this.zoom / gridSize) * gridSize
    endX = startX + (this.canvas.width / this.zoom) + gridSize
    endY = startY + (this.canvas.height / this.zoom) + gridSize

    this.ctx.strokeStyle = '#e0e0e0'
    this.ctx.lineWidth = 1 / this.zoom

    FOR x FROM startX TO endX STEP gridSize
      this.ctx.beginPath()
      this.ctx.moveTo(x, startY)
      this.ctx.lineTo(x, endY)
      this.ctx.stroke()
    END FOR

    FOR y FROM startY TO endY STEP gridSize
      this.ctx.beginPath()
      this.ctx.moveTo(startX, y)
      this.ctx.lineTo(endX, y)
      this.ctx.stroke()
    END FOR

  METHOD drawNode(node: FactoryNode)
    // Node background
    this.ctx.fillStyle = this.getNodeColor(node)
    this.ctx.strokeStyle = this.selectedNodes.has(node.id) ? '#0066ff' : '#333'
    this.ctx.lineWidth = this.selectedNodes.has(node.id) ? 3 : 1

    this.roundRect(node.position.x, node.position.y,
                   node.size.width, node.size.height, 8)
    this.ctx.fill()
    this.ctx.stroke()

    // Node icon
    icon = this.getNodeIcon(node.type)
    this.ctx.drawImage(icon, node.position.x + 10, node.position.y + 10, 24, 24)

    // Node label
    this.ctx.fillStyle = '#333'
    this.ctx.font = '14px Inter'
    this.ctx.fillText(node.label, node.position.x + 40, node.position.y + 28)

    // Status indicator
    this.drawStatusIndicator(node)

    // Input/output ports
    this.drawPorts(node)

    // Queue depth badge
    IF node.queue_depth > 0
      this.drawQueueBadge(node)
    END IF

  METHOD drawConnection(conn: Connection, deltaTime: number)
    sourceNode = this.nodes.get(conn.source_node)
    targetNode = this.nodes.get(conn.target_node)

    sourcePort = this.getPortPosition(sourceNode, conn.source_port, 'output')
    targetPort = this.getPortPosition(targetNode, conn.target_port, 'input')

    // Calculate bezier control points
    dx = targetPort.x - sourcePort.x
    controlPoint1 = {x: sourcePort.x + dx * 0.5, y: sourcePort.y}
    controlPoint2 = {x: targetPort.x - dx * 0.5, y: targetPort.y}

    // Draw belt background
    this.ctx.strokeStyle = conn.color || '#666'
    this.ctx.lineWidth = 6
    this.ctx.lineCap = 'round'
    this.ctx.beginPath()
    this.ctx.moveTo(sourcePort.x, sourcePort.y)
    this.ctx.bezierCurveTo(
      controlPoint1.x, controlPoint1.y,
      controlPoint2.x, controlPoint2.y,
      targetPort.x, targetPort.y
    )
    this.ctx.stroke()

    // Draw animated items on belt
    IF conn.animated AND this.runtime?.status == 'running'
      this.drawBeltItems(conn, sourcePort, targetPort,
                         controlPoint1, controlPoint2, deltaTime)
    END IF

  METHOD drawBeltItems(conn, start, end, cp1, cp2, deltaTime)
    // Animate items along the bezier curve
    itemCount = 3
    speed = 0.0005  // Progress per ms

    FOR i FROM 0 TO itemCount
      // Calculate progress along curve (0-1)
      baseProgress = (i / itemCount)
      animatedProgress = (baseProgress + (performance.now() * speed)) % 1

      // Get position on bezier curve
      pos = this.getBezierPoint(start, cp1, cp2, end, animatedProgress)

      // Draw item
      this.ctx.fillStyle = '#ffd700'
      this.ctx.beginPath()
      this.ctx.arc(pos.x, pos.y, 4, 0, Math.PI * 2)
      this.ctx.fill()
    END FOR

  METHOD getBezierPoint(p0, p1, p2, p3, t)
    // Cubic bezier interpolation
    mt = 1 - t
    mt2 = mt * mt
    mt3 = mt2 * mt
    t2 = t * t
    t3 = t2 * t

    RETURN {
      x: mt3 * p0.x + 3 * mt2 * t * p1.x + 3 * mt * t2 * p2.x + t3 * p3.x,
      y: mt3 * p0.y + 3 * mt2 * t * p1.y + 3 * mt * t2 * p2.y + t3 * p3.y
    }
```

### 2.1.2 Input Handling

```pseudocode
CLASS CanvasInputHandler
  PROPERTY renderer: CanvasRenderer
  PROPERTY dragState: DragState = null
  PROPERTY connectionState: ConnectionState = null

  ENUM DragMode
    NONE, PAN, MOVE_NODE, SELECT_BOX, CONNECT

  METHOD handleMouseDown(event: MouseEvent)
    worldPos = this.screenToWorld(event.clientX, event.clientY)

    // Check if clicking on a port
    port = this.findPortAt(worldPos)
    IF port != null
      this.startConnection(port)
      RETURN
    END IF

    // Check if clicking on a node
    node = this.findNodeAt(worldPos)
    IF node != null
      IF NOT event.shiftKey AND NOT this.renderer.selectedNodes.has(node.id)
        this.renderer.selectedNodes.clear()
      END IF
      this.renderer.selectedNodes.add(node.id)
      this.startNodeDrag(worldPos)
      RETURN
    END IF

    // Clicking on empty space
    IF event.button == 1 OR (event.button == 0 AND event.spaceKey)
      // Middle click or space+click: pan
      this.startPan(event)
    ELSE IF event.button == 0
      // Left click: selection box
      this.renderer.selectedNodes.clear()
      this.startSelectionBox(worldPos)
    END IF

  METHOD handleMouseMove(event: MouseEvent)
    worldPos = this.screenToWorld(event.clientX, event.clientY)

    SWITCH this.dragState.mode
      CASE PAN:
        dx = event.clientX - this.dragState.startScreen.x
        dy = event.clientY - this.dragState.startScreen.y
        this.renderer.pan.x = this.dragState.startPan.x + dx
        this.renderer.pan.y = this.dragState.startPan.y + dy

      CASE MOVE_NODE:
        dx = worldPos.x - this.dragState.startWorld.x
        dy = worldPos.y - this.dragState.startWorld.y
        FOR nodeId IN this.renderer.selectedNodes
          node = this.renderer.nodes.get(nodeId)
          originalPos = this.dragState.originalPositions.get(nodeId)
          node.position.x = this.snapToGrid(originalPos.x + dx)
          node.position.y = this.snapToGrid(originalPos.y + dy)
        END FOR

      CASE SELECT_BOX:
        this.renderer.selectionBox = {
          x: Math.min(this.dragState.startWorld.x, worldPos.x),
          y: Math.min(this.dragState.startWorld.y, worldPos.y),
          width: Math.abs(worldPos.x - this.dragState.startWorld.x),
          height: Math.abs(worldPos.y - this.dragState.startWorld.y)
        }
        this.updateSelectionFromBox()

      CASE CONNECT:
        this.renderer.connectionPreview = {
          start: this.dragState.startPort,
          end: worldPos,
          valid: this.isValidConnectionTarget(worldPos)
        }
    END SWITCH

  METHOD handleMouseUp(event: MouseEvent)
    IF this.dragState.mode == CONNECT
      targetPort = this.findPortAt(this.screenToWorld(event.clientX, event.clientY))
      IF targetPort != null AND this.canConnect(this.dragState.startPort, targetPort)
        this.createConnection(this.dragState.startPort, targetPort)
      END IF
    END IF

    this.dragState = null
    this.renderer.selectionBox = null
    this.renderer.connectionPreview = null

  METHOD handleWheel(event: WheelEvent)
    // Zoom toward mouse position
    mouseWorld = this.screenToWorld(event.clientX, event.clientY)

    zoomFactor = event.deltaY > 0 ? 0.9 : 1.1
    newZoom = Math.max(0.1, Math.min(4.0, this.renderer.zoom * zoomFactor))

    // Adjust pan to keep mouse position stable
    this.renderer.pan.x -= mouseWorld.x * (newZoom - this.renderer.zoom)
    this.renderer.pan.y -= mouseWorld.y * (newZoom - this.renderer.zoom)
    this.renderer.zoom = newZoom

  METHOD screenToWorld(screenX, screenY)
    RETURN {
      x: (screenX - this.renderer.pan.x) / this.renderer.zoom,
      y: (screenY - this.renderer.pan.y) / this.renderer.zoom
    }

  METHOD snapToGrid(value)
    gridSize = 32
    RETURN Math.round(value / gridSize) * gridSize
```

## 2.2 Factory Execution Engine

### 2.2.1 Execution Orchestrator

```pseudocode
CLASS FactoryExecutor
  PROPERTY factory: Factory
  PROPERTY queues: Map<string, WorkItem[]>  // Node ID -> queue
  PROPERTY activeAgents: Map<string, AgentExecution>
  PROPERTY eventBus: EventEmitter
  PROPERTY status: ExecutionStatus

  METHOD start()
    IF this.status == 'running'
      THROW Error("Factory already running")
    END IF

    this.status = 'running'
    this.factory.runtime = {
      status: 'running',
      started_at: new Date().toISOString(),
      items_processed: 0,
      items_failed: 0,
      active_agents: 0,
      token_usage: 0,
      errors: []
    }

    // Initialize queues for all nodes
    FOR node IN this.factory.nodes
      this.queues.set(node.id, [])
    END FOR

    // Start source nodes
    FOR node IN this.factory.nodes
      IF this.isSourceNode(node)
        this.startSourceNode(node)
      END IF
    END FOR

    // Start processing loop
    this.processLoop()

  METHOD stop()
    this.status = 'stopping'

    // Cancel all active agents
    FOR agent IN this.activeAgents.values()
      agent.cancel()
    END FOR

    // Wait for graceful shutdown
    AWAIT this.waitForAgentsToComplete(timeout: 30000)

    this.status = 'stopped'
    this.factory.runtime.status = 'stopped'

  METHOD processLoop()
    WHILE this.status == 'running'
      // Find nodes with items to process
      FOR node IN this.factory.nodes
        IF this.canProcessNode(node)
          item = this.dequeueItem(node.id)
          this.processNode(node, item)
        END IF
      END FOR

      // Small delay to prevent CPU spinning
      AWAIT sleep(10)
    END WHILE

  METHOD canProcessNode(node: FactoryNode)
    // Has items in queue
    IF this.queues.get(node.id).length == 0
      RETURN false
    END IF

    // Not already at capacity
    IF this.isAgentNode(node)
      activeCount = this.countActiveAgentsForNode(node.id)
      IF activeCount >= (node.config.max_concurrent || 1)
        RETURN false
      END IF
    END IF

    // Check resource limits
    IF this.factory.runtime.active_agents >= this.factory.config.max_concurrent_agents
      RETURN false
    END IF

    RETURN true

  METHOD processNode(node: FactoryNode, item: WorkItem)
    // Add trace entry
    item.trace.push({
      node_id: node.id,
      timestamp: new Date().toISOString(),
      action: 'enter',
      duration_ms: 0
    })

    startTime = performance.now()

    TRY
      SWITCH node.type
        CASE 'architect_agent':
        CASE 'developer_agent':
        CASE 'reviewer_agent':
        CASE 'custom_agent':
          result = AWAIT this.executeAgentNode(node, item)

        CASE 'test_runner':
          result = AWAIT this.executeTestRunner(node, item)

        CASE 'git_operations':
          result = AWAIT this.executeGitOps(node, item)

        CASE 'bash_executor':
          result = AWAIT this.executeBash(node, item)

        CASE 'splitter':
          result = this.executeSplitter(node, item)

        CASE 'merger':
          result = this.executeMerger(node, item)

        CASE 'filter':
          result = this.executeFilter(node, item)

        DEFAULT:
          THROW Error("Unknown node type: " + node.type)
      END SWITCH

      // Update trace
      item.trace[item.trace.length - 1].duration_ms = performance.now() - startTime
      item.trace[item.trace.length - 1].action = 'complete'

      // Route output to connected nodes
      this.routeOutput(node, result)

      this.factory.runtime.items_processed++

    CATCH error
      this.handleNodeError(node, item, error)
    END TRY

  METHOD executeAgentNode(node: FactoryNode, item: WorkItem)
    // Create or get Loom thread for this node
    thread = AWAIT this.getOrCreateThread(node)

    // Build prompt from item
    prompt = this.buildPrompt(node, item)

    // Create agent execution
    execution = new AgentExecution({
      node_id: node.id,
      thread: thread,
      model: node.config.model,
      tools: node.config.tools,
      system_prompt: node.config.system_prompt
    })

    this.activeAgents.set(execution.id, execution)
    this.factory.runtime.active_agents++

    // Stream events to UI
    execution.on('text_delta', (text) => {
      this.eventBus.emit('agent_output', {
        node_id: node.id,
        type: 'text',
        content: text
      })
    })

    execution.on('tool_call', (call) => {
      this.eventBus.emit('agent_output', {
        node_id: node.id,
        type: 'tool_call',
        tool: call.name,
        args: call.arguments
      })
    })

    TRY
      // Execute agent
      response = AWAIT execution.run(prompt)

      // Parse structured output
      output = this.parseAgentOutput(node, response)

      // Update token usage
      this.factory.runtime.token_usage += response.usage.total_tokens

      RETURN {
        output_port: output.success ? 'output' : 'failed',
        item: output.item
      }

    FINALLY
      this.activeAgents.delete(execution.id)
      this.factory.runtime.active_agents--
    END TRY

  METHOD routeOutput(node: FactoryNode, result: NodeResult)
    // Find connections from this node's output port
    connections = this.factory.connections.filter(c =>
      c.source_node == node.id AND c.source_port == result.output_port
    )

    FOR conn IN connections
      targetNode = this.factory.nodes.find(n => n.id == conn.target_node)

      // Enqueue item for target node
      this.queues.get(conn.target_node).push(result.item)

      // Emit event for UI animation
      this.eventBus.emit('item_flow', {
        connection_id: conn.id,
        item: result.item
      })

      // Update node queue depth
      targetNode.queue_depth = this.queues.get(conn.target_node).length
    END FOR
```

### 2.2.2 Agent Execution

```pseudocode
CLASS AgentExecution
  PROPERTY id: string
  PROPERTY thread: Thread
  PROPERTY config: AgentConfig
  PROPERTY status: 'pending' | 'running' | 'completed' | 'cancelled' | 'error'
  PROPERTY eventEmitter: EventEmitter
  PROPERTY abortController: AbortController

  METHOD run(prompt: string)
    this.status = 'running'
    this.abortController = new AbortController()

    // Send message to thread
    this.thread.addMessage({
      role: 'user',
      content: prompt
    })

    // Create LLM request
    request = {
      model: this.config.model,
      messages: this.thread.conversation.messages,
      tools: this.config.tools.map(t => t.definition),
      system: this.config.system_prompt,
      max_tokens: this.config.max_tokens || 4096,
      stream: true
    }

    // Call LLM via proxy
    response = AWAIT this.callLlmProxy(request)

    // Process streaming response
    fullResponse = {
      content: '',
      tool_calls: [],
      usage: { input_tokens: 0, output_tokens: 0, total_tokens: 0 }
    }

    FOR AWAIT event IN response.stream
      IF this.abortController.signal.aborted
        THROW new CancelledError()
      END IF

      SWITCH event.type
        CASE 'text_delta':
          fullResponse.content += event.content
          this.eventEmitter.emit('text_delta', event.content)

        CASE 'tool_call_delta':
          this.updateToolCall(fullResponse.tool_calls, event)
          this.eventEmitter.emit('tool_call_delta', event)

        CASE 'completed':
          fullResponse.usage = event.usage
      END SWITCH
    END FOR

    // Execute tool calls if any
    IF fullResponse.tool_calls.length > 0
      toolResults = AWAIT this.executeTools(fullResponse.tool_calls)

      // Add assistant message with tool calls
      this.thread.addMessage({
        role: 'assistant',
        content: fullResponse.content,
        tool_calls: fullResponse.tool_calls
      })

      // Add tool results
      FOR result IN toolResults
        this.thread.addMessage({
          role: 'tool',
          tool_call_id: result.call_id,
          content: JSON.stringify(result.output)
        })
      END FOR

      // Continue conversation for follow-up
      RETURN this.run('')  // Recursive call with empty prompt
    END IF

    // Add final assistant message
    this.thread.addMessage({
      role: 'assistant',
      content: fullResponse.content
    })

    this.status = 'completed'
    RETURN fullResponse

  METHOD executeTools(toolCalls: ToolCall[])
    results = []

    // Execute tools in parallel where safe
    parallelCalls = toolCalls.filter(c => this.isSafeForParallel(c))
    sequentialCalls = toolCalls.filter(c => !this.isSafeForParallel(c))

    // Run parallel tools
    parallelResults = AWAIT Promise.all(
      parallelCalls.map(call => this.executeTool(call))
    )
    results.push(...parallelResults)

    // Run sequential tools
    FOR call IN sequentialCalls
      result = AWAIT this.executeTool(call)
      results.push(result)
    END FOR

    RETURN results

  METHOD executeTool(call: ToolCall)
    tool = this.config.tools.find(t => t.name == call.name)
    IF tool == null
      RETURN { call_id: call.id, error: "Unknown tool: " + call.name }
    END IF

    this.eventEmitter.emit('tool_call', call)

    TRY
      startTime = performance.now()
      output = AWAIT tool.invoke(call.arguments)
      duration = performance.now() - startTime

      this.eventEmitter.emit('tool_result', {
        call_id: call.id,
        success: true,
        duration_ms: duration
      })

      RETURN { call_id: call.id, output: output }

    CATCH error
      this.eventEmitter.emit('tool_result', {
        call_id: call.id,
        success: false,
        error: error.message
      })

      RETURN { call_id: call.id, error: error.message }
    END TRY

  METHOD cancel()
    this.status = 'cancelled'
    this.abortController.abort()
```

## 2.3 State Management

### 2.3.1 Factory Store (Svelte)

```pseudocode
// Using Svelte 5 runes syntax
CLASS FactoryStore
  // Reactive state
  STATE factory: Factory = null
  STATE selectedNodes: Set<string> = new Set()
  STATE selectedConnections: Set<string> = new Set()
  STATE clipboard: ClipboardContent = null
  STATE undoStack: FactorySnapshot[] = []
  STATE redoStack: FactorySnapshot[] = []
  STATE isDirty: boolean = false

  // Derived state
  DERIVED nodeMap = $derived(
    new Map(this.factory?.nodes.map(n => [n.id, n]) ?? [])
  )

  DERIVED connectionMap = $derived(
    new Map(this.factory?.connections.map(c => [c.id, c]) ?? [])
  )

  DERIVED selectedNodeObjects = $derived(
    Array.from(this.selectedNodes).map(id => this.nodeMap.get(id)).filter(Boolean)
  )

  // Actions
  METHOD loadFactory(factoryId: string)
    this.pushUndo()
    response = AWAIT fetch(`/api/factories/${factoryId}`)
    this.factory = AWAIT response.json()
    this.isDirty = false

  METHOD saveFactory()
    IF this.factory == null RETURN

    AWAIT fetch(`/api/factories/${this.factory.id}`, {
      method: 'PUT',
      body: JSON.stringify(this.factory)
    })
    this.isDirty = false

  METHOD addNode(type: NodeType, position: Vector2)
    this.pushUndo()

    node = {
      id: generateId(),
      type: type,
      position: position,
      size: this.getDefaultSize(type),
      label: this.getDefaultLabel(type),
      config: this.getDefaultConfig(type)
    }

    this.factory.nodes.push(node)
    this.isDirty = true

    RETURN node.id

  METHOD deleteSelected()
    IF this.selectedNodes.size == 0 AND this.selectedConnections.size == 0
      RETURN
    END IF

    this.pushUndo()

    // Delete selected connections
    this.factory.connections = this.factory.connections.filter(
      c => !this.selectedConnections.has(c.id)
    )

    // Delete connections to/from selected nodes
    this.factory.connections = this.factory.connections.filter(
      c => !this.selectedNodes.has(c.source_node) AND
           !this.selectedNodes.has(c.target_node)
    )

    // Delete selected nodes
    this.factory.nodes = this.factory.nodes.filter(
      n => !this.selectedNodes.has(n.id)
    )

    this.selectedNodes.clear()
    this.selectedConnections.clear()
    this.isDirty = true

  METHOD addConnection(sourceNode: string, sourcePort: string,
                       targetNode: string, targetPort: string)
    // Validate connection
    IF NOT this.isValidConnection(sourceNode, sourcePort, targetNode, targetPort)
      THROW Error("Invalid connection")
    END IF

    this.pushUndo()

    connection = {
      id: generateId(),
      source_node: sourceNode,
      source_port: sourcePort,
      target_node: targetNode,
      target_port: targetPort
    }

    this.factory.connections.push(connection)
    this.isDirty = true

    RETURN connection.id

  METHOD updateNodeConfig(nodeId: string, config: NodeConfig)
    this.pushUndo()

    node = this.nodeMap.get(nodeId)
    IF node == null RETURN

    node.config = { ...node.config, ...config }
    this.isDirty = true

  METHOD moveNodes(nodeIds: string[], delta: Vector2)
    FOR nodeId IN nodeIds
      node = this.nodeMap.get(nodeId)
      IF node != null
        node.position.x += delta.x
        node.position.y += delta.y
      END IF
    END FOR
    this.isDirty = true

  METHOD copy()
    IF this.selectedNodes.size == 0 RETURN

    // Copy selected nodes
    nodes = this.selectedNodeObjects.map(n => structuredClone(n))

    // Copy connections between selected nodes
    connections = this.factory.connections.filter(c =>
      this.selectedNodes.has(c.source_node) AND
      this.selectedNodes.has(c.target_node)
    ).map(c => structuredClone(c))

    this.clipboard = { nodes, connections }

  METHOD paste(position: Vector2)
    IF this.clipboard == null RETURN

    this.pushUndo()

    // Generate new IDs
    idMap = new Map()
    FOR node IN this.clipboard.nodes
      idMap.set(node.id, generateId())
    END FOR

    // Calculate offset from original position
    minX = Math.min(...this.clipboard.nodes.map(n => n.position.x))
    minY = Math.min(...this.clipboard.nodes.map(n => n.position.y))
    offset = { x: position.x - minX, y: position.y - minY }

    // Create new nodes
    newNodes = this.clipboard.nodes.map(n => ({
      ...structuredClone(n),
      id: idMap.get(n.id),
      position: {
        x: n.position.x + offset.x,
        y: n.position.y + offset.y
      }
    }))

    // Create new connections
    newConnections = this.clipboard.connections.map(c => ({
      ...structuredClone(c),
      id: generateId(),
      source_node: idMap.get(c.source_node),
      target_node: idMap.get(c.target_node)
    }))

    this.factory.nodes.push(...newNodes)
    this.factory.connections.push(...newConnections)

    // Select pasted nodes
    this.selectedNodes = new Set(newNodes.map(n => n.id))
    this.selectedConnections.clear()

    this.isDirty = true

  METHOD undo()
    IF this.undoStack.length == 0 RETURN

    // Save current state to redo
    this.redoStack.push(this.createSnapshot())

    // Restore previous state
    snapshot = this.undoStack.pop()
    this.restoreSnapshot(snapshot)

  METHOD redo()
    IF this.redoStack.length == 0 RETURN

    // Save current state to undo
    this.undoStack.push(this.createSnapshot())

    // Restore next state
    snapshot = this.redoStack.pop()
    this.restoreSnapshot(snapshot)

  PRIVATE METHOD pushUndo()
    this.undoStack.push(this.createSnapshot())
    this.redoStack = []  // Clear redo on new action

    // Limit stack size
    IF this.undoStack.length > 100
      this.undoStack.shift()
    END IF

  PRIVATE METHOD createSnapshot()
    RETURN {
      nodes: structuredClone(this.factory.nodes),
      connections: structuredClone(this.factory.connections),
      selectedNodes: new Set(this.selectedNodes),
      selectedConnections: new Set(this.selectedConnections)
    }

  PRIVATE METHOD restoreSnapshot(snapshot: FactorySnapshot)
    this.factory.nodes = snapshot.nodes
    this.factory.connections = snapshot.connections
    this.selectedNodes = snapshot.selectedNodes
    this.selectedConnections = snapshot.selectedConnections
    this.isDirty = true
```

---

# 3. Architecture

## 3.1 System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              BROWSER CLIENT                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐          │
│  │   Canvas Layer   │  │   UI Components  │  │   State Manager  │          │
│  │   (Pixi.js)      │  │   (Svelte 5)     │  │   (XState)       │          │
│  └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘          │
│           │                     │                     │                     │
│           └─────────────────────┼─────────────────────┘                     │
│                                 │                                           │
│                     ┌───────────▼───────────┐                               │
│                     │    Factory Store      │                               │
│                     │    (Svelte Runes)     │                               │
│                     └───────────┬───────────┘                               │
│                                 │                                           │
│           ┌─────────────────────┼─────────────────────┐                     │
│           │                     │                     │                     │
│  ┌────────▼────────┐  ┌────────▼────────┐  ┌────────▼────────┐            │
│  │   API Client    │  │   WS Client     │  │   SSE Client    │            │
│  │   (REST)        │  │   (Real-time)   │  │   (Streaming)   │            │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘            │
│           │                     │                     │                     │
└───────────┼─────────────────────┼─────────────────────┼─────────────────────┘
            │                     │                     │
            │    HTTP/REST        │    WebSocket        │    SSE
            │                     │                     │
┌───────────┼─────────────────────┼─────────────────────┼─────────────────────┐
│           │                     │                     │                     │
│           ▼                     ▼                     ▼                     │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         LOOM SERVER                                  │   │
│  │                                                                      │   │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐        │   │
│  │  │  Factory API   │  │  WS Handler    │  │  SSE Handler   │        │   │
│  │  │  /api/factory  │  │  /ws/factory   │  │  /sse/factory  │        │   │
│  │  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘        │   │
│  │          │                   │                   │                  │   │
│  │          └───────────────────┼───────────────────┘                  │   │
│  │                              │                                      │   │
│  │                   ┌──────────▼──────────┐                          │   │
│  │                   │  Factory Service    │                          │   │
│  │                   │                     │                          │   │
│  │                   │  - CRUD operations  │                          │   │
│  │                   │  - Execution engine │                          │   │
│  │                   │  - Event routing    │                          │   │
│  │                   └──────────┬──────────┘                          │   │
│  │                              │                                      │   │
│  │    ┌─────────────────────────┼─────────────────────────┐           │   │
│  │    │                         │                         │           │   │
│  │    ▼                         ▼                         ▼           │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │   │
│  │  │   Thread     │  │    LLM       │  │   Weaver     │            │   │
│  │  │   Service    │  │   Service    │  │   Service    │            │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘            │   │
│  │         │                 │                 │                     │   │
│  └─────────┼─────────────────┼─────────────────┼─────────────────────┘   │
│            │                 │                 │                         │
│  LOOM SERVER                 │                 │                         │
└────────────┼─────────────────┼─────────────────┼─────────────────────────┘
             │                 │                 │
             ▼                 ▼                 ▼
┌────────────────────┐  ┌──────────────┐  ┌──────────────────┐
│      SQLite        │  │  LLM APIs    │  │   Kubernetes     │
│    (Factories,     │  │  (Anthropic, │  │   (Weaver Pods)  │
│     Threads)       │  │   OpenAI)    │  │                  │
└────────────────────┘  └──────────────┘  └──────────────────┘
```

## 3.2 Component Architecture

### 3.2.1 Frontend Components

```
src/
├── lib/
│   ├── components/
│   │   ├── canvas/
│   │   │   ├── Canvas.svelte              # Main canvas component
│   │   │   ├── CanvasRenderer.ts          # Pixi.js rendering
│   │   │   ├── CanvasInput.ts             # Input handling
│   │   │   ├── Grid.ts                    # Background grid
│   │   │   └── Minimap.svelte             # Minimap overlay
│   │   │
│   │   ├── nodes/
│   │   │   ├── Node.svelte                # Base node component
│   │   │   ├── NodePort.svelte            # Input/output port
│   │   │   ├── NodeConfig.svelte          # Configuration panel
│   │   │   ├── AgentNode.svelte           # Agent-specific node
│   │   │   ├── ToolNode.svelte            # Tool-specific node
│   │   │   ├── FlowNode.svelte            # Flow control node
│   │   │   └── SourceNode.svelte          # Source node
│   │   │
│   │   ├── connections/
│   │   │   ├── Connection.svelte          # Belt connection
│   │   │   ├── ConnectionPreview.svelte   # Drag preview
│   │   │   └── ItemAnimation.ts           # Animated items
│   │   │
│   │   ├── palette/
│   │   │   ├── NodePalette.svelte         # Node type selector
│   │   │   ├── PaletteCategory.svelte     # Category group
│   │   │   └── PaletteItem.svelte         # Draggable item
│   │   │
│   │   ├── toolbar/
│   │   │   ├── Toolbar.svelte             # Main toolbar
│   │   │   ├── PlayControls.svelte        # Start/stop/pause
│   │   │   ├── ZoomControls.svelte        # Zoom in/out/fit
│   │   │   └── UndoRedo.svelte            # Undo/redo buttons
│   │   │
│   │   ├── panels/
│   │   │   ├── PropertiesPanel.svelte     # Selected node config
│   │   │   ├── ExecutionPanel.svelte      # Runtime status
│   │   │   ├── LogPanel.svelte            # Execution logs
│   │   │   └── BlueprintPanel.svelte      # Blueprint browser
│   │   │
│   │   └── dialogs/
│   │       ├── SaveDialog.svelte          # Save factory
│   │       ├── LoadDialog.svelte          # Load factory
│   │       ├── ExportDialog.svelte        # Export blueprint
│   │       └── SettingsDialog.svelte      # Factory settings
│   │
│   ├── stores/
│   │   ├── factory.svelte.ts              # Factory state (runes)
│   │   ├── execution.svelte.ts            # Runtime state
│   │   ├── ui.svelte.ts                   # UI state
│   │   └── user.svelte.ts                 # User preferences
│   │
│   ├── machines/
│   │   ├── canvas.machine.ts              # Canvas state machine
│   │   ├── execution.machine.ts           # Execution state machine
│   │   └── connection.machine.ts          # Connection state machine
│   │
│   ├── services/
│   │   ├── api.ts                         # REST API client
│   │   ├── websocket.ts                   # WebSocket client
│   │   ├── sse.ts                         # SSE client
│   │   └── storage.ts                     # Local storage
│   │
│   ├── types/
│   │   ├── factory.ts                     # Factory types
│   │   ├── node.ts                        # Node types
│   │   ├── connection.ts                  # Connection types
│   │   └── execution.ts                   # Runtime types
│   │
│   └── utils/
│       ├── geometry.ts                    # Vector math, bezier
│       ├── validation.ts                  # Connection validation
│       ├── serialization.ts               # JSON/YAML conversion
│       └── id.ts                          # ID generation
│
├── routes/
│   ├── +layout.svelte                     # App layout
│   ├── +page.svelte                       # Home/dashboard
│   ├── factory/
│   │   ├── +page.svelte                   # Factory list
│   │   ├── [id]/
│   │   │   └── +page.svelte               # Factory editor
│   │   └── new/
│   │       └── +page.svelte               # Create factory
│   └── blueprints/
│       ├── +page.svelte                   # Blueprint library
│       └── [id]/
│           └── +page.svelte               # Blueprint detail
│
└── app.html                               # HTML template
```

### 3.2.2 Backend Components

```
crates/
├── loom-server-factory/
│   ├── src/
│   │   ├── lib.rs                         # Module exports
│   │   ├── api/
│   │   │   ├── mod.rs
│   │   │   ├── routes.rs                  # HTTP routes
│   │   │   ├── handlers.rs                # Request handlers
│   │   │   └── dto.rs                     # Data transfer objects
│   │   │
│   │   ├── service/
│   │   │   ├── mod.rs
│   │   │   ├── factory_service.rs         # CRUD operations
│   │   │   ├── execution_service.rs       # Runtime management
│   │   │   └── blueprint_service.rs       # Blueprint operations
│   │   │
│   │   ├── executor/
│   │   │   ├── mod.rs
│   │   │   ├── orchestrator.rs            # Execution orchestrator
│   │   │   ├── agent_executor.rs          # Agent node execution
│   │   │   ├── tool_executor.rs           # Tool node execution
│   │   │   └── flow_executor.rs           # Flow control execution
│   │   │
│   │   ├── nodes/
│   │   │   ├── mod.rs
│   │   │   ├── agent_nodes.rs             # Agent node types
│   │   │   ├── tool_nodes.rs              # Tool node types
│   │   │   ├── flow_nodes.rs              # Flow control nodes
│   │   │   ├── source_nodes.rs            # Source nodes
│   │   │   └── sink_nodes.rs              # Sink nodes
│   │   │
│   │   ├── types/
│   │   │   ├── mod.rs
│   │   │   ├── factory.rs                 # Factory types
│   │   │   ├── node.rs                    # Node types
│   │   │   ├── connection.rs              # Connection types
│   │   │   ├── work_item.rs               # Work item types
│   │   │   └── blueprint.rs               # Blueprint types
│   │   │
│   │   └── db/
│   │       ├── mod.rs
│   │       ├── factory_repo.rs            # Factory persistence
│   │       └── blueprint_repo.rs          # Blueprint persistence
│   │
│   └── Cargo.toml
│
├── loom-server-factory-ws/
│   ├── src/
│   │   ├── lib.rs
│   │   ├── handler.rs                     # WebSocket handler
│   │   ├── protocol.rs                    # Message protocol
│   │   └── broadcast.rs                   # Event broadcasting
│   │
│   └── Cargo.toml
│
└── loom-factory-types/
    ├── src/
    │   ├── lib.rs
    │   └── ... (shared types)
    └── Cargo.toml
```

## 3.3 Database Schema

```sql
-- Factory definitions
CREATE TABLE factories (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    version INTEGER NOT NULL DEFAULT 1,
    owner_id TEXT NOT NULL REFERENCES users(id),
    visibility TEXT NOT NULL DEFAULT 'private',
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL,
    deleted_at TEXT,

    -- Serialized factory data
    canvas_state JSON NOT NULL,
    nodes JSON NOT NULL,
    connections JSON NOT NULL,
    config JSON NOT NULL,

    -- Full JSON for schema evolution
    full_json JSON NOT NULL
);

CREATE INDEX idx_factories_owner ON factories(owner_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_factories_visibility ON factories(visibility) WHERE deleted_at IS NULL;

-- Factory execution history
CREATE TABLE factory_executions (
    id TEXT PRIMARY KEY,
    factory_id TEXT NOT NULL REFERENCES factories(id),
    started_at TEXT NOT NULL,
    ended_at TEXT,
    status TEXT NOT NULL,  -- 'running', 'completed', 'failed', 'cancelled'

    -- Metrics
    items_processed INTEGER NOT NULL DEFAULT 0,
    items_failed INTEGER NOT NULL DEFAULT 0,
    token_usage INTEGER NOT NULL DEFAULT 0,

    -- Error info if failed
    error_message TEXT,
    error_node_id TEXT
);

CREATE INDEX idx_executions_factory ON factory_executions(factory_id);
CREATE INDEX idx_executions_status ON factory_executions(status) WHERE ended_at IS NULL;

-- Execution events (for replay and debugging)
CREATE TABLE execution_events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    execution_id TEXT NOT NULL REFERENCES factory_executions(id),
    timestamp TEXT NOT NULL,
    event_type TEXT NOT NULL,
    node_id TEXT,
    item_id TEXT,
    payload JSON NOT NULL
);

CREATE INDEX idx_events_execution ON execution_events(execution_id);

-- Blueprints (shareable factory templates)
CREATE TABLE blueprints (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    version TEXT NOT NULL,  -- Semver
    author_id TEXT NOT NULL REFERENCES users(id),
    visibility TEXT NOT NULL DEFAULT 'private',
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL,

    -- Tags for discovery
    tags JSON NOT NULL DEFAULT '[]',

    -- Factory template (without runtime state)
    factory_template JSON NOT NULL,

    -- Parameters for customization
    parameters JSON NOT NULL DEFAULT '[]',

    -- Stats
    downloads INTEGER NOT NULL DEFAULT 0,
    stars INTEGER NOT NULL DEFAULT 0
);

CREATE INDEX idx_blueprints_author ON blueprints(author_id);
CREATE INDEX idx_blueprints_visibility ON blueprints(visibility);
CREATE INDEX idx_blueprints_downloads ON blueprints(downloads DESC);

-- Blueprint stars (likes)
CREATE TABLE blueprint_stars (
    blueprint_id TEXT NOT NULL REFERENCES blueprints(id),
    user_id TEXT NOT NULL REFERENCES users(id),
    created_at TEXT NOT NULL,
    PRIMARY KEY (blueprint_id, user_id)
);
```

## 3.4 API Design

### 3.4.1 REST Endpoints

```yaml
# Factory CRUD
POST   /api/v1/factories                    # Create factory
GET    /api/v1/factories                    # List user's factories
GET    /api/v1/factories/:id                # Get factory
PUT    /api/v1/factories/:id                # Update factory
DELETE /api/v1/factories/:id                # Soft delete factory

# Factory execution
POST   /api/v1/factories/:id/start          # Start execution
POST   /api/v1/factories/:id/stop           # Stop execution
POST   /api/v1/factories/:id/pause          # Pause execution
POST   /api/v1/factories/:id/resume         # Resume execution
GET    /api/v1/factories/:id/status         # Get runtime status

# Factory history
GET    /api/v1/factories/:id/executions     # List executions
GET    /api/v1/factories/:id/executions/:eid # Get execution details
GET    /api/v1/factories/:id/executions/:eid/events # Get execution events

# Blueprints
POST   /api/v1/blueprints                   # Create blueprint
GET    /api/v1/blueprints                   # List blueprints (with search)
GET    /api/v1/blueprints/:id               # Get blueprint
PUT    /api/v1/blueprints/:id               # Update blueprint
DELETE /api/v1/blueprints/:id               # Delete blueprint
POST   /api/v1/blueprints/:id/instantiate   # Create factory from blueprint
POST   /api/v1/blueprints/:id/star          # Star blueprint
DELETE /api/v1/blueprints/:id/star          # Unstar blueprint

# Import/Export
POST   /api/v1/factories/:id/export         # Export as blueprint
POST   /api/v1/import                       # Import from JSON/YAML
```

### 3.4.2 WebSocket Protocol

```typescript
// Client -> Server messages
type ClientMessage =
  | { type: 'subscribe'; factory_id: string }
  | { type: 'unsubscribe'; factory_id: string }
  | { type: 'node_update'; node_id: string; changes: Partial<NodeConfig> }
  | { type: 'manual_input'; node_id: string; item: WorkItem }
  | { type: 'pause_node'; node_id: string }
  | { type: 'resume_node'; node_id: string }

// Server -> Client messages
type ServerMessage =
  | { type: 'factory_state'; factory: Factory }
  | { type: 'node_status'; node_id: string; status: NodeStatus }
  | { type: 'item_enqueued'; node_id: string; item: WorkItem }
  | { type: 'item_processed'; node_id: string; item: WorkItem; result: NodeResult }
  | { type: 'item_flow'; connection_id: string; item: WorkItem }
  | { type: 'agent_output'; node_id: string; event: AgentEvent }
  | { type: 'execution_error'; node_id: string; error: string }
  | { type: 'metrics_update'; metrics: FactoryMetrics }
```

### 3.4.3 SSE Events (for agent streaming)

```typescript
// Endpoint: GET /api/v1/factories/:id/stream

// Events
event: agent_text
data: {"node_id": "abc", "content": "Let me analyze..."}

event: agent_tool_call
data: {"node_id": "abc", "tool": "read_file", "args": {"path": "/src/main.rs"}}

event: agent_tool_result
data: {"node_id": "abc", "tool": "read_file", "success": true, "duration_ms": 45}

event: node_complete
data: {"node_id": "abc", "output_port": "output", "item_id": "xyz"}

event: item_flow
data: {"connection_id": "conn1", "item_id": "xyz"}

event: error
data: {"node_id": "abc", "message": "Rate limit exceeded", "retrying": true}
```

## 3.5 State Machine Definitions

### 3.5.1 Canvas State Machine (XState)

```typescript
import { createMachine, assign } from 'xstate';

const canvasMachine = createMachine({
  id: 'canvas',
  initial: 'idle',
  context: {
    selectedNodes: [] as string[],
    selectedConnections: [] as string[],
    dragStart: null as Vector2 | null,
    connectionStart: null as PortRef | null,
    selectionBox: null as Rect | null,
  },
  states: {
    idle: {
      on: {
        MOUSE_DOWN_NODE: {
          target: 'draggingNodes',
          actions: 'selectNode',
        },
        MOUSE_DOWN_PORT: {
          target: 'connecting',
          actions: 'startConnection',
        },
        MOUSE_DOWN_CANVAS: [
          {
            target: 'panning',
            guard: 'isMiddleClick',
          },
          {
            target: 'selecting',
            actions: 'clearSelection',
          },
        ],
        DELETE_KEY: {
          actions: 'deleteSelected',
        },
        COPY: {
          actions: 'copySelected',
        },
        PASTE: {
          actions: 'pasteClipboard',
        },
        UNDO: {
          actions: 'undo',
        },
        REDO: {
          actions: 'redo',
        },
      },
    },

    draggingNodes: {
      on: {
        MOUSE_MOVE: {
          actions: 'updateNodePositions',
        },
        MOUSE_UP: {
          target: 'idle',
          actions: 'snapToGrid',
        },
      },
    },

    connecting: {
      on: {
        MOUSE_MOVE: {
          actions: 'updateConnectionPreview',
        },
        MOUSE_UP_PORT: [
          {
            target: 'idle',
            guard: 'isValidConnection',
            actions: 'createConnection',
          },
          {
            target: 'idle',
            actions: 'cancelConnection',
          },
        ],
        MOUSE_UP: {
          target: 'idle',
          actions: 'cancelConnection',
        },
      },
    },

    selecting: {
      on: {
        MOUSE_MOVE: {
          actions: 'updateSelectionBox',
        },
        MOUSE_UP: {
          target: 'idle',
          actions: 'finalizeSelection',
        },
      },
    },

    panning: {
      on: {
        MOUSE_MOVE: {
          actions: 'updatePan',
        },
        MOUSE_UP: {
          target: 'idle',
        },
      },
    },
  },
});
```

### 3.5.2 Execution State Machine

```typescript
const executionMachine = createMachine({
  id: 'execution',
  initial: 'stopped',
  context: {
    factory: null as Factory | null,
    runtime: null as FactoryRuntime | null,
    errors: [] as RuntimeError[],
  },
  states: {
    stopped: {
      on: {
        START: {
          target: 'starting',
          guard: 'hasFactory',
        },
      },
    },

    starting: {
      invoke: {
        src: 'startExecution',
        onDone: {
          target: 'running',
          actions: 'initRuntime',
        },
        onError: {
          target: 'error',
          actions: 'setError',
        },
      },
    },

    running: {
      on: {
        PAUSE: 'paused',
        STOP: 'stopping',
        NODE_ERROR: {
          actions: 'addError',
        },
        ITEM_PROCESSED: {
          actions: 'updateMetrics',
        },
      },
    },

    paused: {
      on: {
        RESUME: 'running',
        STOP: 'stopping',
      },
    },

    stopping: {
      invoke: {
        src: 'stopExecution',
        onDone: {
          target: 'stopped',
          actions: 'clearRuntime',
        },
        onError: {
          target: 'error',
          actions: 'setError',
        },
      },
    },

    error: {
      on: {
        RETRY: 'starting',
        RESET: {
          target: 'stopped',
          actions: 'clearError',
        },
      },
    },
  },
});
```

---

# 4. Refinement

## 4.1 Performance Optimizations

### 4.1.1 Canvas Rendering

| Optimization | Technique | Impact |
|--------------|-----------|--------|
| Spatial indexing | R-tree for node/connection lookup | O(log n) hit testing |
| View frustum culling | Only render visible nodes | 10x fewer draw calls |
| Connection caching | Pre-compute bezier paths | Avoid recalculation |
| Dirty rect rendering | Only redraw changed areas | 50% fewer pixels |
| Object pooling | Reuse animation sprites | Reduce GC pressure |
| Level-of-detail | Simplify at low zoom | Faster render at overview |

### 4.1.2 State Management

| Optimization | Technique | Impact |
|--------------|-----------|--------|
| Immutable updates | Structural sharing | Efficient change detection |
| Selective reactivity | Fine-grained subscriptions | Fewer re-renders |
| Debounced saves | Coalesce rapid changes | Reduce API calls |
| Virtualized lists | Only render visible items | Handle 1000s of items |
| Web Workers | Offload heavy computation | Keep UI responsive |

### 4.1.3 Network Efficiency

| Optimization | Technique | Impact |
|--------------|-----------|--------|
| Delta sync | Send only changes | 90% smaller payloads |
| Compression | gzip/brotli responses | 70% smaller transfers |
| Connection pooling | Reuse HTTP/2 connections | Faster requests |
| Request batching | Combine multiple updates | Fewer round trips |
| Optimistic updates | Apply locally first | Instant feedback |

## 4.2 Security Considerations

### 4.2.1 Input Validation

```rust
// All user input must be validated
impl FactoryService {
    pub fn create_factory(&self, input: CreateFactoryInput) -> Result<Factory> {
        // Validate name
        validate_name(&input.name)?;

        // Validate node configurations
        for node in &input.nodes {
            self.validate_node_config(node)?;
        }

        // Validate connections
        self.validate_connections(&input.nodes, &input.connections)?;

        // Check resource limits
        self.check_quota(&input)?;

        // ...
    }

    fn validate_node_config(&self, node: &NodeInput) -> Result<()> {
        match node.node_type {
            NodeType::BashExecutor => {
                // Validate command doesn't contain dangerous patterns
                let config: BashConfig = serde_json::from_value(node.config.clone())?;
                validate_bash_command(&config.command)?;
            }
            NodeType::CustomAgent => {
                // Validate system prompt length
                let config: CustomAgentConfig = serde_json::from_value(node.config.clone())?;
                if config.system_prompt.len() > MAX_SYSTEM_PROMPT_LENGTH {
                    return Err(Error::ValidationError("System prompt too long".into()));
                }
            }
            // ...
        }
        Ok(())
    }
}
```

### 4.2.2 Authorization

```rust
// ABAC policy for factory operations
impl FactoryPolicy {
    pub fn can_view(&self, user: &User, factory: &Factory) -> bool {
        match factory.visibility {
            Visibility::Public => true,
            Visibility::Organization => {
                user.organization_id == factory.owner_organization_id
            }
            Visibility::Private => {
                user.id == factory.owner_id
            }
        }
    }

    pub fn can_edit(&self, user: &User, factory: &Factory) -> bool {
        user.id == factory.owner_id ||
        self.is_org_admin(user, factory.owner_organization_id)
    }

    pub fn can_execute(&self, user: &User, factory: &Factory) -> bool {
        self.can_view(user, factory) &&
        user.has_permission(Permission::ExecuteFactory)
    }
}
```

### 4.2.3 Resource Limits

```rust
// Per-user resource limits
struct ResourceLimits {
    max_factories: u32,
    max_nodes_per_factory: u32,
    max_concurrent_executions: u32,
    max_tokens_per_hour: u64,
    max_weaver_hours_per_day: f64,
}

impl ExecutionService {
    pub async fn start_execution(&self, factory_id: &str) -> Result<()> {
        let user = self.get_current_user()?;
        let limits = self.get_user_limits(&user)?;

        // Check concurrent execution limit
        let active_count = self.count_active_executions(&user.id).await?;
        if active_count >= limits.max_concurrent_executions {
            return Err(Error::ResourceLimitExceeded(
                "Max concurrent executions reached".into()
            ));
        }

        // Check token budget
        let tokens_used = self.get_tokens_used_this_hour(&user.id).await?;
        if tokens_used >= limits.max_tokens_per_hour {
            return Err(Error::ResourceLimitExceeded(
                "Hourly token limit reached".into()
            ));
        }

        // ...
    }
}
```

## 4.3 Error Handling

### 4.3.1 Error Taxonomy

```rust
#[derive(Debug, thiserror::Error)]
pub enum FactoryError {
    // Validation errors (400)
    #[error("Invalid factory configuration: {0}")]
    ValidationError(String),

    #[error("Invalid node configuration for {node_type}: {message}")]
    InvalidNodeConfig { node_type: String, message: String },

    #[error("Invalid connection: {0}")]
    InvalidConnection(String),

    // Authorization errors (403)
    #[error("Not authorized to {action} factory {factory_id}")]
    NotAuthorized { action: String, factory_id: String },

    // Not found errors (404)
    #[error("Factory not found: {0}")]
    FactoryNotFound(String),

    #[error("Blueprint not found: {0}")]
    BlueprintNotFound(String),

    // Conflict errors (409)
    #[error("Factory version conflict: expected {expected}, got {actual}")]
    VersionConflict { expected: u64, actual: u64 },

    // Resource limit errors (429)
    #[error("Resource limit exceeded: {0}")]
    ResourceLimitExceeded(String),

    // Execution errors (500)
    #[error("Node execution failed: {node_id}: {message}")]
    NodeExecutionFailed { node_id: String, message: String },

    #[error("Agent error: {0}")]
    AgentError(#[from] AgentError),

    // Infrastructure errors
    #[error("Database error: {0}")]
    DatabaseError(#[from] sqlx::Error),

    #[error("LLM service error: {0}")]
    LlmError(#[from] LlmError),
}
```

### 4.3.2 Retry Policies

```rust
impl ExecutionOrchestrator {
    async fn execute_with_retry<F, T>(
        &self,
        node: &FactoryNode,
        operation: F,
    ) -> Result<T>
    where
        F: Fn() -> Future<Output = Result<T>>,
    {
        let policy = self.factory.config.retry_policy.clone();
        let mut attempts = 0;
        let mut delay = policy.initial_delay_ms;

        loop {
            attempts += 1;

            match operation().await {
                Ok(result) => return Ok(result),
                Err(e) if self.is_retryable(&e) && attempts < policy.max_attempts => {
                    // Log retry
                    tracing::warn!(
                        node_id = %node.id,
                        attempt = attempts,
                        error = %e,
                        delay_ms = delay,
                        "Retrying node execution"
                    );

                    // Emit event for UI
                    self.emit_event(ExecutionEvent::RetryScheduled {
                        node_id: node.id.clone(),
                        attempt: attempts,
                        delay_ms: delay,
                        error: e.to_string(),
                    });

                    // Wait with jitter
                    let jitter = rand::thread_rng().gen_range(0..delay / 4);
                    tokio::time::sleep(Duration::from_millis(delay + jitter)).await;

                    // Exponential backoff
                    delay = std::cmp::min(
                        delay * policy.backoff_multiplier as u64,
                        policy.max_delay_ms
                    );
                }
                Err(e) => {
                    // Final failure
                    return Err(FactoryError::NodeExecutionFailed {
                        node_id: node.id.clone(),
                        message: format!("Failed after {} attempts: {}", attempts, e),
                    });
                }
            }
        }
    }

    fn is_retryable(&self, error: &FactoryError) -> bool {
        matches!(error,
            FactoryError::LlmError(LlmError::RateLimited { .. }) |
            FactoryError::LlmError(LlmError::Timeout) |
            FactoryError::LlmError(LlmError::ServiceUnavailable) |
            FactoryError::AgentError(AgentError::ToolTimeout { .. })
        )
    }
}
```

## 4.4 Testing Strategy

### 4.4.1 Unit Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use proptest::prelude::*;

    // Property-based test for connection validation
    proptest! {
        #[test]
        fn connection_validation_is_symmetric(
            source_type in arb_node_type(),
            target_type in arb_node_type(),
            source_port in arb_port_name(),
            target_port in arb_port_name(),
        ) {
            let source = create_test_node(source_type);
            let target = create_test_node(target_type);

            let result = validate_connection(&source, &source_port, &target, &target_port);

            // Connection validity should be deterministic
            let result2 = validate_connection(&source, &source_port, &target, &target_port);
            prop_assert_eq!(result.is_ok(), result2.is_ok());
        }
    }

    #[test]
    fn test_splitter_round_robin() {
        let splitter = SplitterNode::new(SplitterConfig {
            mode: SplitterMode::RoundRobin,
            output_count: 3,
        });

        let items: Vec<_> = (0..9).map(|i| create_test_item(i)).collect();
        let results: Vec<_> = items.iter().map(|i| splitter.process(i)).collect();

        // Items should be distributed evenly
        assert_eq!(results.iter().filter(|r| r.output_port == "output_0").count(), 3);
        assert_eq!(results.iter().filter(|r| r.output_port == "output_1").count(), 3);
        assert_eq!(results.iter().filter(|r| r.output_port == "output_2").count(), 3);
    }
}
```

### 4.4.2 Integration Tests

```rust
#[tokio::test]
async fn test_simple_pipeline_execution() {
    let app = TestApp::new().await;

    // Create a simple factory: IssueInbox -> DeveloperAgent -> PRCreator
    let factory = app.create_factory(json!({
        "name": "Test Pipeline",
        "nodes": [
            {
                "id": "inbox",
                "type": "manual_input",
                "position": {"x": 0, "y": 0},
                "config": {}
            },
            {
                "id": "developer",
                "type": "developer_agent",
                "position": {"x": 200, "y": 0},
                "config": {
                    "model": "claude-3-5-sonnet",
                    "tools": ["read_file", "edit_file"]
                }
            },
            {
                "id": "logger",
                "type": "logger",
                "position": {"x": 400, "y": 0},
                "config": {"format": "json"}
            }
        ],
        "connections": [
            {"source_node": "inbox", "source_port": "output", "target_node": "developer", "target_port": "input"},
            {"source_node": "developer", "source_port": "output", "target_node": "logger", "target_port": "input"}
        ]
    })).await;

    // Start execution
    app.start_factory(&factory.id).await;

    // Inject a test item
    app.inject_item(&factory.id, "inbox", json!({
        "type": "issue",
        "payload": {
            "title": "Add hello world function",
            "body": "Please add a function that prints hello world"
        }
    })).await;

    // Wait for processing
    let events = app.wait_for_events(&factory.id, 10, Duration::from_secs(30)).await;

    // Verify execution
    assert!(events.iter().any(|e| e.event_type == "agent_text"));
    assert!(events.iter().any(|e| e.event_type == "node_complete" && e.node_id == "developer"));
    assert!(events.iter().any(|e| e.event_type == "node_complete" && e.node_id == "logger"));
}
```

### 4.4.3 E2E Tests

```typescript
// Playwright test for canvas interactions
import { test, expect } from '@playwright/test';

test('can create and connect nodes', async ({ page }) => {
  await page.goto('/factory/new');

  // Open node palette
  await page.click('[data-testid="add-node-button"]');

  // Drag architect agent to canvas
  const architectItem = page.locator('[data-testid="palette-item-architect_agent"]');
  const canvas = page.locator('[data-testid="factory-canvas"]');

  await architectItem.dragTo(canvas, {
    targetPosition: { x: 200, y: 200 }
  });

  // Verify node was created
  const architectNode = page.locator('[data-testid="node-architect_agent"]');
  await expect(architectNode).toBeVisible();

  // Add developer agent
  await page.click('[data-testid="add-node-button"]');
  const developerItem = page.locator('[data-testid="palette-item-developer_agent"]');
  await developerItem.dragTo(canvas, {
    targetPosition: { x: 400, y: 200 }
  });

  // Connect them
  const outputPort = page.locator('[data-testid="node-architect_agent"] [data-testid="port-output"]');
  const inputPort = page.locator('[data-testid="node-developer_agent"] [data-testid="port-input"]');

  await outputPort.dragTo(inputPort);

  // Verify connection
  const connection = page.locator('[data-testid="connection"]');
  await expect(connection).toBeVisible();
});

test('can start and stop execution', async ({ page }) => {
  await page.goto('/factory/test-factory-id');

  // Start execution
  await page.click('[data-testid="start-button"]');

  // Verify running state
  await expect(page.locator('[data-testid="status-running"]')).toBeVisible();

  // Inject test item
  await page.click('[data-testid="inject-item-button"]');
  await page.fill('[data-testid="item-content"]', 'Test input');
  await page.click('[data-testid="inject-submit"]');

  // Wait for processing indicator
  await expect(page.locator('[data-testid="node-working"]')).toBeVisible();

  // Stop execution
  await page.click('[data-testid="stop-button"]');
  await expect(page.locator('[data-testid="status-stopped"]')).toBeVisible();
});
```

---

# 5. Completion

## 5.1 Implementation Phases

### Phase 1: Foundation (4 weeks)

#### Week 1-2: Core Canvas
- [ ] Canvas rendering with Pixi.js
- [ ] Grid and pan/zoom
- [ ] Node rendering (static)
- [ ] Basic input handling

#### Week 3-4: Node System
- [ ] Node drag-and-drop
- [ ] Port system
- [ ] Connection drawing
- [ ] Selection system

**Deliverable**: Interactive canvas with static nodes and connections

### Phase 2: State & Persistence (3 weeks)

#### Week 5-6: State Management
- [ ] Factory store (Svelte runes)
- [ ] Undo/redo system
- [ ] Copy/paste
- [ ] Auto-save

#### Week 7: Backend API
- [ ] Factory CRUD endpoints
- [ ] Database schema
- [ ] Blueprint system

**Deliverable**: Persistent factories with version control

### Phase 3: Execution Engine (4 weeks)

#### Week 8-9: Core Executor
- [ ] Execution orchestrator
- [ ] Queue management
- [ ] Basic node execution
- [ ] Event system

#### Week 10-11: Agent Integration
- [ ] Agent node execution
- [ ] Thread management
- [ ] Tool execution
- [ ] Streaming responses

**Deliverable**: Working agent-based pipelines

### Phase 4: Real-time Features (3 weeks)

#### Week 12: WebSocket Integration
- [ ] WebSocket server
- [ ] Client subscription
- [ ] Event broadcasting

#### Week 13-14: Live UI
- [ ] Real-time status updates
- [ ] Animated belt items
- [ ] Live agent output
- [ ] Metrics dashboard

**Deliverable**: Real-time execution visualization

### Phase 5: Advanced Features (4 weeks)

#### Week 15-16: Flow Control
- [ ] Splitter/merger nodes
- [ ] Filter nodes
- [ ] Buffer/rate limiter
- [ ] Conditional routing

#### Week 17-18: External Integrations
- [ ] Issue inbox (GitHub/Jira)
- [ ] PR creator
- [ ] Notifications
- [ ] Webhooks

**Deliverable**: Full-featured workflow automation

### Phase 6: Polish (2 weeks)

#### Week 19: UX Polish
- [ ] Keyboard shortcuts
- [ ] Accessibility
- [ ] Error messages
- [ ] Loading states

#### Week 20: Documentation & Testing
- [ ] User documentation
- [ ] API documentation
- [ ] E2E test coverage
- [ ] Performance optimization

**Deliverable**: Production-ready release

## 5.2 Milestones & Deliverables

| Milestone | Week | Deliverables |
|-----------|------|--------------|
| M1: Canvas MVP | 4 | Interactive canvas, static nodes |
| M2: Persistence | 7 | Save/load factories, blueprints |
| M3: Execution | 11 | Working agent pipelines |
| M4: Real-time | 14 | Live execution visualization |
| M5: Advanced | 18 | Flow control, integrations |
| M6: Release | 20 | Production-ready system |

## 5.3 Success Metrics

### User Engagement
- [ ] 100+ factories created in first month
- [ ] 50+ blueprints shared
- [ ] 70% 7-day retention

### Performance
- [ ] 60fps canvas with 100 nodes
- [ ] <2s initial load time
- [ ] <100ms node placement

### Reliability
- [ ] 99.9% execution success rate
- [ ] <1% data loss incidents
- [ ] <5min mean recovery time

### Quality
- [ ] >80% test coverage
- [ ] <10 P1 bugs at launch
- [ ] >4.0 user satisfaction score

## 5.4 Risk Mitigation

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Canvas performance issues | High | Medium | Early benchmarking, LOD system |
| WebSocket scalability | High | Medium | Connection pooling, Redis pub/sub |
| Agent execution costs | Medium | High | Token budgets, caching |
| Complex state bugs | Medium | Medium | XState, extensive testing |
| User confusion | Medium | Medium | Onboarding tutorial, templates |

## 5.5 Future Enhancements (Post-Launch)

### Short-term (Q1)
- Mobile-responsive canvas
- Collaborative editing
- Version history viewer
- Custom node SDK

### Medium-term (Q2)
- Multi-agent coordination
- AI-suggested optimizations
- Performance analytics
- Blueprint marketplace

### Long-term (Q3+)
- Visual debugging/replay
- Self-optimizing factories
- Cross-factory orchestration
- Enterprise features (SSO, audit logs)

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| **Factory** | A complete workflow definition with nodes, connections, and configuration |
| **Node** | A processing unit in the factory (agent, tool, flow control) |
| **Connection** | A data flow link between two nodes (visualized as a belt) |
| **Work Item** | A unit of data flowing through the factory |
| **Blueprint** | A shareable factory template |
| **Weaver** | A remote execution environment (K8s pod) |
| **Port** | An input or output point on a node |
| **Belt** | Visual representation of a connection with animated items |

## Appendix B: Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Delete` / `Backspace` | Delete selected |
| `Ctrl+C` | Copy |
| `Ctrl+V` | Paste |
| `Ctrl+Z` | Undo |
| `Ctrl+Shift+Z` | Redo |
| `Ctrl+A` | Select all |
| `Escape` | Clear selection |
| `Space` (hold) | Pan mode |
| `+` / `-` | Zoom in/out |
| `0` | Fit to view |
| `Ctrl+S` | Save |

## Appendix C: Color Scheme

| Element | Color | Hex |
|---------|-------|-----|
| Canvas background | Light gray | `#f5f5f5` |
| Grid lines | Medium gray | `#e0e0e0` |
| Node background | White | `#ffffff` |
| Node border | Dark gray | `#333333` |
| Selected border | Blue | `#0066ff` |
| Belt (issue) | Yellow | `#ffd700` |
| Belt (code) | Blue | `#4a90d9` |
| Belt (review) | Purple | `#9b59b6` |
| Belt (test) | Green | `#2ecc71` |
| Status: idle | Gray | `#95a5a6` |
| Status: working | Blue | `#3498db` |
| Status: error | Red | `#e74c3c` |
| Status: success | Green | `#27ae60` |
