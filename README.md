# 🤖 FREYABOT OS — System Technical Documentation

> **Version**: 8.0.0 | **Author**: Lucas Germán Neme | **License**: Proprietary  
> **Main Stack**: Rust (Backend) + Next.js/React (GUI) + Electron (Native Shell)

---

## 📋 Table of Contents

1. [System Overview](#1-system-overview)
2. [High-Level Architecture](#2-high-level-architecture)
3. [System Components](#3-system-components)
   - 3.1 [freyabot-core (Rust Backend)](#31-freyabot-core-rust-backend)
   - 3.2 [gui (Next.js + Electron Frontend)](#32-gui-nextjs--electron-frontend)
   - 3.3 [ghost-bridge (Legacy CLI Bridge)](#33-ghost-bridge-legacy-cli-bridge)
4. [Backend Modules in Detail](#4-backend-modules-in-detail)
   - 4.1 [main.rs — Entry Point and Bootstrap](#41-mainrs--entry-point-and-bootstrap)
   - 4.2 [orchestrator.rs — LLM Session Orchestrator](#42-orchestratorrs--llm-session-orchestrator)
   - 4.3 [cli_bridge.rs — Dual Agentic Engine](#43-cli_bridgers--dual-agentic-engine)
   - 4.4 [context_builder.rs — Dynamic Prompt Builder](#44-context_builderrs--dynamic-prompt-builder)
   - 4.5 [agent_tools.rs — Tool Arsenal](#45-agent_toolsrs--tool-arsenal)
   - 4.6 [backend/agent_coordinator.rs — Swarm Coordinator](#46-backendagent_coordinatorrs--swarm-coordinator)
   - 4.7 [backend/worker_engine.rs — Worker Engine](#47-backendworker_enginers--worker-engine)
   - 4.8 [telemetry.rs — Hardware Sensor and Metrics](#48-telemetryrs--hardware-sensor-and-metrics)
   - 4.9 [system.rs — Configuration and Persistence](#49-systemrs--configuration-and-persistence)
   - 4.10 [session_store.rs — Session Persistence](#410-session_storers--session-persistence)
   - 4.11 [self_healing.rs — Self-Recovery](#411-self_healingrs--self-recovery)
   - 4.12 [validation.rs — Project Validator](#412-validationrs--project-validator)
   - 4.13 [watcher.rs — Filesystem Sentinel](#413-watcherrs--filesystem-sentinel)
   - 4.14 [brain/ — Mycelial Network (Episodic Memory)](#414-brain--mycelial-network-episodic-memory)
   - 4.15 [tools/web_search.rs — Web Search](#415-toolsweb_searchrs--web-search)
   - 4.16 [projects.rs — Multi-Project Management](#416-projectsrs--multi-project-management)
   - 4.17 [filesystem.rs — Filesystem Operations](#417-filesystemrs--filesystem-operations)
   - 4.18 [tasks.rs — Kanban System](#418-tasksrs--kanban-system)
   - 4.19 [models.rs — Data Models](#419-modelsrs--data-models)
5. [Frontend GUI in Detail](#5-frontend-gui-in-detail)
   - 5.1 [Main Components](#51-main-components)
   - 5.2 [Routing and Layout System](#52-routing-and-layout-system)
   - 5.3 [Electron Shell (main.js)](#53-electron-shell-mainjs)
6. [System Protocols](#6-system-protocols)
   - 6.1 [Sovereign Protocol](#61-sovereign-protocol)
   - 6.2 [Agentic Loop](#62-agentic-loop)
   - 6.3 [Anti-Amputation Protocol](#63-anti-amputation-protocol)
   - 6.4 [Auto-Healing (Self-Healing Loop)](#64-auto-healing-self-healing-loop)
   - 6.5 [Agentic Shielding (Tail Appendment)](#65-agentic-shielding-tail-appendment)
   - 6.6 [Deterministic State Machine](#66-deterministic-state-machine)
7. [Complete REST API](#7-complete-rest-api)
8. [LLM Provider System](#8-llm-provider-system)
9. [Project Directory Structure](#9-project-directory-structure)
10. [Internal Conventions and Standards](#10-internal-conventions-and-standards)
11. [End-to-End Data Flow](#11-end-to-end-data-flow)
12. [Persistence System](#12-persistence-system)
13. [Security and Sandboxing](#13-security-and-sandboxing)

---

## 1. System Overview

**Freyabot OS** is an AI agent orchestration operating system oriented towards software development. It works as a native Windows desktop application that combines:

- A **high-performance backend** written in Rust (Axum) that manages LLM communication, tool execution, and hardware telemetry.
- A **rich frontend** built with Next.js + React + TailwindCSS, featuring a Monaco code editor, native PTY terminals (xterm.js), file explorer, Kanban board, and a real-time streaming chat panel.
- A **native Electron shell** that packages everything into a portable Windows application with a custom `app://` protocol, Rust sidecar, and native PTY terminals.

The system operates under the **"Sovereign Bunker"** paradigm: an autonomous workstation that can work with both local LLMs (LM Studio, Ollama) and cloud providers (Google Gemini, Groq, SambaNova, OpenCode Zen) without mandatory dependency on any third-party service.

---

## 2. High-Level Architecture

```
┌───────────────────────────────────────────────────────────────────────┐
│                         ELECTRON SHELL (main.js)                     │
│  ┌──────────────────┐    ┌──────────────────────────────────────┐    │
│  │  Rust Sidecar     │    │   Next.js Frontend (React/TSX)      │    │
│  │  (freyabot-core)  │←──│                                      │    │
│  │                   │──→│   Chat | Editor | Terminal | Kanban   │    │
│  │  Port: 45000+     │SSE│   File Explorer | Metrics | Swarm    │    │
│  └────────┬──────────┘   └──────────────┬───────────────────────┘    │
│           │                              │                            │
│           │ HTTP REST API                │ IPC (Electron)             │
│           ▼                              ▼                            │
│  ┌──────────────────┐    ┌──────────────────────────────────────┐    │
│  │  LLM Provider     │    │  node-pty (Native Terminal)          │    │
│  │  Multi-Cloud Hub  │    │  Monaco Editor                       │    │
│  │  (Gemini/Groq/    │    │  File System Bridge                  │    │
│  │   SambaNova/Local) │    └──────────────────────────────────────┘    │
│  └──────────────────┘                                                │
└───────────────────────────────────────────────────────────────────────┘
```

### Main Communication Flow

1. **User → Frontend (Next.js)**: Interaction via Chat, Editor, or Terminal.
2. **Frontend → Backend (Rust)**: HTTP REST API at `http://127.0.0.1:{PORT}`.
3. **Backend → LLM**: SSE streaming towards the selected LLM provider.
4. **LLM → Backend**: Response with text and/or tool calls.
5. **Backend → Tools**: Execution of `write_file`, `execute_command`, etc.
6. **Backend → Frontend**: SSE Events with text chunks, state changes, and alerts.
7. **Frontend → User**: Real-time chat rendering + reactive filesystem, terminal, and Kanban updates.

---

## 3. System Components

### 3.1 freyabot-core (Rust Backend)

**Path**: `freyabot-core/`  
**Language**: Rust (Edition 2021)  
**HTTP Framework**: Axum 0.7  
**Runtime**: Tokio (full features)

#### Key Dependencies:

| Crate | Usage |
|-------|-------|
| `axum` | HTTP REST + SSE Server |
| `tokio` | Async runtime + broadcast channels |
| `serde` / `serde_json` | JSON serialization |
| `reqwest` | HTTP client for LLM APIs |
| `notify` | Native File System Watcher |
| `sysinfo` / `wmi` / `windows` | Hardware telemetry (CPU, RAM, GPU/VRAM) |
| `fastembed` | Embedding engine (ONNX Runtime, AllMiniLmL6V2) |
| `scraper` | Web scraping (DuckDuckGo HTML) |
| `tree-sitter` / `tree-sitter-rust` | AST parsing for skeletons |
| `git2` | Native Git integration |
| `regex` | ANSI cleanup and parsing |
| `chrono` | RFC3339 timestamps |
| `uuid` | Session ID generation |
| `rfd` | Native OS dialogs |

#### Modules:

```
src/
├── main.rs                 # Bootstrap, Router, Global State
├── orchestrator.rs         # LLM session orchestrator (SSE endpoint)
├── cli_bridge.rs           # Dual Agentic Engine (API + CLI Passthrough)
├── context_builder.rs      # Dynamic System Prompt builder
├── agent_tools.rs          # Tool implementations + OpenAI Schemas
├── agent_tools_schemas.rs  # Additional schemas
├── models.rs               # Data models (Task, Settings, Workspace, Metrics)
├── system.rs               # Configuration, persistence, path resolution
├── projects.rs             # Multi-project management
├── session_store.rs        # Interrupted session persistence
├── self_healing.rs         # Stuck task self-recovery
├── validation.rs           # Multi-language validator (cargo check, tsc, etc.)
├── telemetry.rs            # Hardware sensor (CPU/RAM/GPU/VRAM via DXGI+WMI)
├── watcher.rs              # File System Watcher with debounce
├── filesystem.rs           # File tree API and read/write operations
├── tasks.rs                # Kanban system (CRUD + SSE streaming)
├── parser.rs               # AST skeleton generator (tree-sitter)
├── engine_manager.rs       # Local LLM engine management
├── context_manager.rs      # Context cache (open files, git state)
├── knowledge_manager.rs    # Knowledge rule assimilation
├── clipboard.rs            # Virtual clipboard (file cut/copy/paste)
├── webhooks.rs             # Webhook receiver (Sentry error tracker)
├── mcp.rs                  # Model Context Protocol (stub)
├── logging.rs              # Persistent LLM request logging
├── brain/
│   ├── mod.rs              # Brain root module
│   ├── embeddings.rs       # Embedding engine (fastembed/ONNX)
│   └── vector_store.rs     # Vector database (Mycelium)
├── backend/
│   ├── mod.rs              # Agentic backend root module
│   ├── agent_coordinator.rs # Swarm Coordinator
│   └── worker_engine.rs    # Worker execution engine
├── tools/
│   ├── mod.rs              # Tools root module
│   ├── file_ops.rs         # Atomic file operations
│   ├── send_message.rs     # Inter-agent communication
│   ├── spawn_agent.rs      # Worker agent creation
│   └── web_search.rs       # Web search engine (DDG + Tavily)
└── bin/
    └── test_wmi.rs         # WMI diagnostic utility
```

### 3.2 gui (Next.js + Electron Frontend)

**Path**: `gui/`  
**Framework**: Next.js 14.2.35 + React 18  
**Styling**: TailwindCSS 4.2  
**Editor**: Monaco Editor (via `@monaco-editor/react`)  
**Terminal**: xterm.js + xterm-addon-fit  
**Animations**: Framer Motion  
**Icons**: Lucide React  
**Markdown**: react-markdown + remark-gfm  
**Validation**: Zod  
**Shell**: Electron 31.x  

#### Frontend Structure:

```
gui/src/
├── app/
│   ├── page.tsx              # Main page (51KB — entire dashboard)
│   ├── layout.tsx            # Root layout with typography
│   ├── globals.css           # Global styles
│   ├── login/                # Login page
│   ├── editor-standalone/    # Standalone Monaco editor
│   ├── preview-standalone/   # Live Preview standalone
│   └── fonts/                # Local fonts
├── components/
│   ├── FileExplorer.tsx      # File explorer (44KB)
│   ├── MainWorkspace.tsx     # Main workspace (Editor + Tabs)
│   ├── SettingsModal.tsx     # Settings modal (30KB)
│   ├── LivePreview.tsx       # Live web application preview
│   ├── ProjectLauncher.tsx   # Project launcher
│   ├── SwarmDashboard.tsx    # Worker Swarm dashboard
│   ├── MetricsPanel.tsx      # System metrics panel
│   ├── SetupWizardModal.tsx  # Initial setup wizard
│   ├── BottomPanels.tsx      # Bottom panels (Terminal + Logs)
│   ├── MenuBar.tsx           # Top menu bar
│   ├── SidebarNavigation.tsx # Side navigation
│   ├── MemoryInsight.tsx     # Mycelial Network visualization
│   ├── TaskBoard.tsx         # Kanban board
│   ├── XTermTerminal.tsx     # xterm.js terminal component
│   ├── XTermLog.tsx          # xterm.js log component
│   ├── GenericModal.tsx      # Reusable generic modal
│   ├── AgentStatus.tsx       # Agent status indicator
│   ├── BunkerShield.tsx      # Bunker visual shield
│   ├── RadarDashboard.tsx    # Radar-type dashboard
│   ├── RadarView.tsx         # Radar view
│   └── TurnoCard.tsx         # Shift card
├── hooks/
│   ├── useSettings.ts        # Persistent settings hook
│   └── useWorkspace.ts       # Workspace state hook
├── services/
│   └── auth.service.ts       # Authentication service
├── lib/
│   ├── config.ts             # Dynamic configuration (API URL)
│   ├── ipc.ts                # IPC Bridge (Qt/Electron/Mock)
│   └── schemas.ts            # Zod schemas
├── types/
│   ├── auth.types.ts         # Authentication types
│   └── product.types.ts      # Product types
└── utils/
    └── format.ts             # Formatting utilities
```

### 3.3 ghost-bridge (Legacy CLI Bridge)

**Path**: `ghost-bridge/`  
**Type**: Auxiliary Rust binary  
**Purpose**: Bridge between the backend and the OpenCode CLI (Node.js wrapper). Handles three operating modes:

1. **Telemetry Mode**: Fast hardware metrics reading (CPU, RAM, VRAM via WMI).
2. **Oneshot Mode**: Single OpenCode task execution via STDIN.
3. **Daemon Mode**: JSON STDIN listening loop for continuous execution.

---

## 4. Backend Modules in Detail

### 4.1 main.rs — Entry Point and Bootstrap

The `main.rs` file defines the system boot sequence:

#### Bootstrap Sequence:

```
[0] Sovereign argument parsing (--port, --data-dir)
    ↓
[0b] Logging system initialization (tracing)
    ↓
[1] Pre-boot Self-Healing (recover stuck tasks)
    ↓
[1b] Stale session cleanup (>24h)
    ↓
[2] Broadcast channel configuration:
    - tx/rx: Main SSE channel (1000 buffer)
    - telemetry_tx/rx: Telemetry channel (1000 buffer)
    ↓
[2.1] File Watcher for last active project
    ↓
[2.5] Worker cleanup cycle (every 30s)
    ↓
[3] Background Telemetry loop (every 5s):
    - CPU/RAM reading
    - One-time Swarm recalibration based on VRAM
    ↓
[4] CORS configuration + Axum Router + Bind
    ↓
[5] Sentinel registration (.freyabot/api.url)
    ↓
[6] Server with Graceful Shutdown (Ctrl+C cleans sentinel)
```

#### Global State (`AppState`):

```rust
pub struct AppState {
    pub tx: broadcast::Sender<String>,           // Main SSE channel
    pub telemetry_tx: broadcast::Sender<String>,  // Telemetry channel
    pub clipboard: Mutex<Option<ClipboardState>>, // Virtual clipboard
    pub download_progress: AtomicU64,             // Download progress
    pub context_cache: Arc<RwLock<ContextCache>>, // File/git cache
    pub coordinator: Arc<AgentCoordinator>,        // Swarm Coordinator
}
```

### 4.2 orchestrator.rs — LLM Session Orchestrator

The orchestrator is the brain of the LLM interaction system. It manages:

#### Main Endpoint: `POST /api/v1/orchestrate/ask`

**Payload**:
```json
{
  "message": "string",
  "role": "string (forge|architect)",
  "history": [{"role": "user|assistant", "content": "..."}],
  "model_id": "string (optional)",
  "agent_mode": "string (optional)",
  "reasoning_effort": "string (optional)",
  "mcp_enabled": "boolean (optional)",
  "active_file": "string (optional)",
  "workspace_dir": "string (optional)",
  "sop": "string (optional)",
  "ui_mode": "plan|build (optional)",
  "ui_effort": "low|med|high|one (optional)",
  "session_id": "string (optional)"
}
```

#### Workspace Awareness:

The orchestrator automatically detects the workspace context:

- **PLAN Mode + Empty Project** → Activates `architect_prd` SOP (generates PRD, TRD, Budget).
- **PLAN Mode + Project with code** → Activates `feature_extension` SOP (impact analysis).
- **BUILD Mode** → No architect SOP, focus on direct execution.

#### Processing Flow:

```
[1] Workspace Awareness (detect effective role)
    ↓
[2] System Prompt construction (context_builder)
    ↓
[3] Conversational context assembly
    ↓
[4] Preventive Persistence (pre-flight snapshot)
    ↓
[5] CLI Bridge dispatch (ask_opencode_cli_stream)
    ↓
[6] SSE stream to frontend:
    - session_init → chunk → status → error → plan_ready
    - status_update → fs_change → tasks_sync
    - session_persisted → action_completed → [DONE]
```

#### Resumable Session:

The system supports interrupted session resumption via:

- `POST /api/v1/orchestrate/resume` — Stitching by overlap (last 400 chars).
- `GET /api/v1/orchestrate/sessions` — Lists interrupted sessions.

#### Rollback:

- `POST /api/v1/orchestrate/rollback` — Cleans files, MVP folder, and Kanban.

### 4.3 cli_bridge.rs — Dual Agentic Engine

This is the largest and most complex module in the system (~83KB, ~1400 lines). It implements the complete **Agentic Loop**.

#### Two Operating Modes:

##### CLI Mode (OpenCode Passthrough):
- Executes `node opencode run` as a child process.
- Injects the complete prompt via a temporary file.
- Reads stdout/stderr in streaming.
- Runs Auto-Healing post-execution (up to 3 retries).
- Cleans temporary files after each attempt.

##### Native Mode (HTTP API / LM Studio):
- Direct connection to the configured LLM endpoint.
- Tool support (OpenAI-compatible Function Calling).
- Multi-iteration agentic loop with intelligent stopping.

#### Intensity Configuration (Effort):

| Effort | Max Steps | Max Actions | Auto-Healing | Usage |
|--------|-----------|-------------|--------------|-------|
| `low` | 6 | 6 | ❌ | Quick queries |
| `one` | 1 | 1 | ❌ | Interactive mode |
| `med` | 25 | 15 | ✅ | Standard production |
| `high` | 40 | 30 | ✅ | Extended autonomy |

#### Multi-Tool Engine:

Inspired by the StreamingToolExecutor pattern, it supports:

- **Parallel tool calls**: Multiple tools in a single LLM response.
- **Streaming argument accumulation**: Function call arguments arrive fragmented via SSE; the bridge reconstructs them in an indexed buffer by `index`.
- **Robust normalization** (`normalize_tool_arguments`): Handles escaped JSON strings, triple-quoted, direct objects, etc.

#### Anti-Amputation Protocol (FREIA-85):

When the LLM responds with `finish_reason: "length"` (token limit):

1. Accumulates the text generated so far.
2. Injects a continuation message: `"Continue from where you left off"`.
3. Retries up to `max_continuations` (5) times.
4. The assistant text is joined with automatic **stitching**.

#### Build Mode Auto-Detection:

If PRD files exist in `mvp/`:
- **Build Enforcement** is activated: The LLM is forced to delegate tasks to Workers via `spawn_worker` + `dispatch_worker`.
- The Architect does not write code directly; it acts as an orchestrator.

### 4.4 context_builder.rs — Dynamic Prompt Builder

Generates the System Prompt with semantic XML structure:

#### Prompt Structure (PromptTicket):

```xml
# FREYABOT OS — SESSION DIRECTIVE

<role>
  * ROLE: FORGE operating under Freyabot OS v8.
  * WORKSPACE: `C:/Projects/myproject`
  * DETECTED LANGUAGE: React/TSX
  * DIRECTIVE: All paths must be relative to the workspace root.
</role>

<context>
  <active_file>
    path: `src/App.tsx`
    lang: React/TSX
    ```
    [active file content, maximum 3000 chars]
    ```
  </active_file>
  * GIT STATE: [branch: main] | 3 files modified
  * HARDWARE: CPU: i7-12700H | RAM: 32768 MB | GPU: RTX 3060 | OS: windows x86_64
</context>

<memory>
  #1 [project-x] (2026-03-15, React, score: 92%): "Use shadcn/ui for components"
</memory>

<constraints>
  *** EXECUTION LAWS ***
  - LAW 1: Read before write.
  - LAW 2: No dependency installation.
  - LAW 3: Maximum 3 new files per task.
  - LAW 4: Files >50 lines → use edit_file_atomic.
  
  <project_memory>
    [Content from FREIA.md or CODING_STANDARDS.md if exists]
  </project_memory>
</constraints>

<pipeline>
  1. READ → 2. PLAN → 3. EXECUTE → 4. VALIDATE
</pipeline>

<thought>
  Before generating code, open a <thought> block with analysis.
</thought>

<examples>
  [Context-relevant few-shot examples]
</examples>
```

#### Contextual Intelligence Functions:

- **`detect_language()`**: Detects language/framework by file extension.
- **`needs_hardware_context()`**: Injects hardware specs only if the prompt mentions GPU/VRAM/ML.
- **`fetch_dynamic_system_profile()`**: Cross-platform CPU/RAM/GPU reading.
- **`load_dynamic_sop()`**: Loads Standard Operating Procedures from `.freyabot/team/sops/`.
- **Episodic Memory (Mycelium)**: Searches for semantically similar memories and injects them if score > 85%.
- **Few-Shot Examples**: Loads examples from `.freyabot/examples/` filtered by relevance.

#### Project Memory Hierarchy:

```
1. .freia/memory/FREIA.md              (New standard)
2. .freyabot/team/CODING_STANDARDS.md  (Project-local)
3. {FREYABOT_ROOT}/.freyabot/team/CODING_STANDARDS.md (Global)
```

### 4.5 agent_tools.rs — Tool Arsenal

Implements the tools that the LLM can invoke:

#### Available Tools (12 tools):

| Tool | Description | Restriction |
|------|-------------|-------------|
| `read_file` | Reads file with line numbers. >1000 lines → AST Skeleton | Architect + Forge |
| `write_file` | Creates/overwrites file. Sanitizes absolute paths | Architect + Forge |
| `edit_file_atomic` | Replaces exact text block (find/replace) | Architect + Forge |
| `list_files` | Lists directories (ignores node_modules, target, dotfiles) | Architect + Forge |
| `execute_command` | Executes terminal (whitelist: npm, npx, cargo, git, rustc, node, bun) | Forge only |
| `create_task` | Creates a Kanban task | Architect only |
| `bulk_create_tasks` | Creates multiple tasks at once | Architect only |
| `update_task_status` | Updates task status (PENDING→BUILDING→DONE→FAILED) | Architect + Forge |
| `bootstrap_project_docs` | Atomically generates PRD + TRD + Budget | Architect only |
| `web_search_validator` | Web search to validate technologies | Architect + Forge |
| `spawn_worker` | Creates a specialized worker in the Swarm | Architect only |
| `dispatch_worker` | Sends task to a worker and waits for result | Architect only |

#### `execute_command` Security:

- **Command whitelist**: `npm`, `npx`, `bun`, `cargo`, `git`, `rustc`, `node`.
- **Adaptive timeout**: 60s normal, 180s for install/build.
- **Kill on drop**: Child process is killed if the future is cancelled.
- **Smart Truncate**: LLM receives only the first 500 + last 2000 chars of output.
- **Real-time streaming**: Stdout/stderr are emitted to the frontend via telemetry.
- **Persistent logging**: Each execution is saved to `.freyabot/logs/terminal.log`.

#### Path Sanitization:

```rust
// All paths are resolved relative to the active project
let root = crate::projects::get_active_project().await;
let path = root.join(path_str);

// Path traversal protection
if !full_path.starts_with(&root) {
    return "[ERROR] Path not allowed";
}
```

#### Task Management:

- Tasks are stored in `.freyabot/team/tasks.json`.
- **Deduplication**: If a task with an existing title is created, it gets updated.
- **STOP_AGENT sentinel**: When marked as DONE/FAILED, `[STOP_AGENT]` is emitted to stop the agentic loop.
- **Random IDs**: Format `TASK-XXXXXX` (6 alphanumeric chars).

### 4.6 backend/agent_coordinator.rs — Swarm Coordinator

The **Iron Coordinator** manages the fleet of concurrent Workers.

#### Swarm Architecture:

```
┌──────────────────────────────────────────┐
│           AGENT COORDINATOR              │
│                                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │Worker #1│  │Worker #2│  │Worker #3│ │
│  │ (coder) │  │ (tester)│  │(designer│ │
│  └────┬────┘  └────┬────┘  └────┬────┘ │
│       │            │            │       │
│       ▼            ▼            ▼       │
│  ┌─────────────────────────────────────┐│
│  │    Semaphore (Concurrency Queue)    ││
│  │    Max: 4 slots (calibratable)     ││
│  └─────────────────────────────────────┘│
└──────────────────────────────────────────┘
```

#### Dynamic Calibration:

The number of Workers is calibrated based on hardware:

| RAM | VRAM | Workers |
|-----|------|---------|
| < 4 GB | - | 1 |
| 4-8 GB | - | 2 |
| 8-16 GB | - | 3 |
| ≥ 16 GB | - | 4 |
| - | ≥ 6 GB | +1 bonus |

#### Worker Lifecycle:

```
spawn_worker(worker_id, role)
    ↓
Worker Loop (tokio::spawn):
    - Waits for WorkerMessage::Task {content, result_tx}
    - Creates WorkerEngine and executes task
    - Sends result via oneshot channel
    - Waits for next task or WorkerMessage::Terminate
    ↓
Automatic cleanup on inactivity (>600s)
```

#### Communication:

- **mpsc channel** (32 buffer): Sends tasks and signals to Workers.
- **oneshot channel**: Returns results to the Architect.
- **Timeout**: 600 seconds maximum per Worker task.

### 4.7 backend/worker_engine.rs — Worker Engine

Each Worker runs a **mini agentic loop** with its own context:

#### Differences from the Main Loop:

| Feature | Main Loop | Worker Engine |
|---------|-----------|---------------|
| Streaming | ✅ SSE | ❌ Non-streaming |
| Max Steps | 25-40 | 10 |
| Arsenal | 12 tools | 5 tools (safe subset) |
| UI Output | Text chunks | Telemetry only |
| Recursion | ✅ Can spawn workers | ❌ Forbidden |

#### Worker-Safe Arsenal:

Only `read_file`, `write_file`, `execute_command`, `edit_file_atomic`, `list_files`.

**Intentionally excluded**: `spawn_worker`, `dispatch_worker`, `create_task`, `bulk_create_tasks`, `bootstrap_project_docs`, `web_search_validator` — to prevent infinite recursion and agent creation loops.

#### Provider Dispatcher:

The Worker Engine replicates the main loop's LLM provider selection logic, including configurable support via `worker_model` in settings.

### 4.8 telemetry.rs — Hardware Sensor and Metrics

#### HardwareSensor:

Cross-platform reading with three layers:

1. **sysinfo**: CPU usage, RAM (cross-platform).
2. **DXGI (DirectX)**: GPU name, total dedicated VRAM (native Windows via COM).
3. **WMI**: VRAM in use, GPU engine utilization (Windows, periodic queries).

#### Exposed Metrics (`GET /api/v1/system/metrics/live`):

```json
{
  "os_name": "Windows 11 Pro",
  "cpu_name": "12th Gen Intel i7-12700H",
  "gpu_name": "NVIDIA GeForce RTX 3060 Laptop GPU",
  "cpu": 15.3,
  "ram_used_gb": 12.4,
  "ram_total_gb": 31.7,
  "ram_percent": 39.1,
  "gpu_usage": 23,
  "vram_dedicated_used": 2.1,
  "vram_dedicated_total": 6.0,
  "vram_dedicated_phys": 6.0,
  "vram_dedicated_percent": 35.0,
  "vram_shared_used": 0.8,
  "vram_shared_total": 15.9,
  "gpu_offline": false,
  "missions": 12,
  "success_rate": 83.3,
  "savings": 30.0
}
```

#### Business Logic:

- **Missions**: Count of completed tasks (DONE) in the Kanban.
- **Success Rate**: `(DONE / total_tasks) * 100`.
- **Savings**: `$2.50 × done_tasks` (automated task savings estimate).

#### Broadcast Telemetry:

```rust
pub fn broadcast_telemetry(tx, channel, data) {
    let msg = json!({ "channel": channel, "data": data }).to_string();
    tx.send(msg);
}
```

Telemetry channels: `core`, `system`, `worker`, `terminal`, `hardware_metrics`.

### 4.9 system.rs — Configuration and Persistence

#### Path Resolution:

##### `project_root()`:
1. `FREYABOT_ROOT` environment variable (highest priority).
2. Development mode: `CARGO_MANIFEST_DIR/../`.
3. Production mode: executable directory.

##### `data_dir()`:
1. Override via `--data-dir` (absolute priority).
2. Portable Mode: If `.portable` exists → `.freyabot/data/`.
3. Standard: `%LOCALAPPDATA%/freyabot/`.
4. Fallback: `%USERPROFILE%/Documents/freyabot/`.
5. Emergency: local `.freyabot/data/`.

#### JSON Persistence:

- **Reading with triple migration**: Searches in `data_dir` → migrates from `Roaming` → migrates from `root`.
- **Atomic writing**: `write → .tmp` → `rename → .json` (prevents corruption).
- **Shallow Merge**: PATCH updates perform a shallow merge without deleting unspecified fields.

#### Configuration Files:

| File | Content |
|------|---------|
| `settings.json` | Enabled models, API keys, LLM endpoints, project history |
| `workspace.json` | Workspace layout, open tabs, panel sizes, last project |
| `session.json` | Authentication token, user data |

### 4.10 session_store.rs — Session Persistence

#### SessionSnapshot:

```rust
struct SessionSnapshot {
    session_id: String,
    role: String,
    model: String,
    system_prompt: String,
    user_prompt: String,
    messages: Vec<Value>,
    accumulated_text: String,
    step: usize,
    continuation_count: usize,
    timestamp: String,        // RFC3339
    status: SessionStatus,    // Active | Interrupted | Completed
}
```

#### Persistence Flow:

1. **Pre-flight**: A snapshot is saved before starting the LLM call.
2. **Periodic**: `accumulated_text` is updated every 2 seconds.
3. **ChunkAck**: Persisted on each acknowledgment event.
4. **Completion**: On finish, the session file is **deleted** (not marked as Completed, it's removed).

#### Per-Project Storage:

Independent from system sessions, each project has its own conversation history:

- Path: `{project_root}/.freia/memory/session.json`
- Automatically saved when writing `workspace.json` with messages.

### 4.11 self_healing.rs — Self-Recovery

Runs at system startup (`run_self_healing()`):

#### Logic:

1. Reads `.freyabot/team/tasks.json`.
2. Searches for tasks with `BUILDING` status (possible previous interruption).
3. Executes `git status --porcelain`:
   - **With Git changes** → Marks as `DONE` (assumes work was completed).
   - **No changes** → Marks as `REVIEW` (caution, needs human verification).

### 4.12 validation.rs — Project Validator

Automatic project type detection and silent validation:

| Detected File | Command | Timeout |
|---------------|---------|---------|
| `Cargo.toml` | `cargo check` | 20s |
| `tsconfig.json` | `npx tsc --noEmit` | 20s |
| `package.json` | `node --check index.js` | 20s |
| `requirements.txt` / `pyproject.toml` | `python -m py_compile` | 20s |

Used by the CLI Bridge's Auto-Healing to verify that LLM changes didn't break the build.

### 4.13 watcher.rs — Filesystem Sentinel

Uses the `notify` crate (RecommendedWatcher) with:

- **Poll interval**: 500ms.
- **Recursive mode**: Watches the entire project directory.
- **Debounce**: 200ms of silence before emitting an event (avoids bursts).
- **Task detection**: If the change is in `tasks.json`, emits `tasks_sync` instead of `fs_change`.
- **Dynamic**: Restarts when switching projects.

### 4.14 brain/ — Mycelial Network (Episodic Memory)

#### Embedding Engine (`embeddings.rs`):

- **Model**: AllMiniLmL6V2 (fastembed/ONNX Runtime).
- **Dimensions**: 384.
- **Hardware**: Strictly CPU (does not touch GPU/VRAM).
- **Lazy initialization**: Loaded on first call (~80MB weights).
- **Thread-safe**: Singleton with `OnceLock<Mutex<TextEmbedding>>`.

#### Vector Store (`vector_store.rs`):

- **Zero-Dependency Implementation**: Persistent JSON + brute-force cosine search.
- **Path**: `{data_dir}/brain/micelio.json`.
- **Search**: O(n) with cosine similarity — sub-millisecond latency for hundreds of memories.
- **Memory Structure**:

```json
{
  "id": "uuid",
  "project_id": "freyabotos",
  "date": "2026-03-15",
  "technology": "React",
  "content": "Use shadcn/ui for UI components",
  "vector": [0.123, -0.456, ...] // 384 dims
}
```

- **Deduplication**: By `id` on storage.
- **Atomic writing**: tmp + rename.

### 4.15 tools/web_search.rs — Web Search

Two providers with automatic fallback:

#### DuckDuckGo (Lite/HTML):
- No API key required.
- Scraping from `html.duckduckgo.com/html/`.
- Chrome User-Agent to avoid blocks.
- Extracts top 5 results (title, URL, snippet).

#### Tavily (Premium):
- Requires `tavily_api_key` in settings.
- REST API with `search_depth: "advanced"`.
- Maximum 5 results with extended content.

#### Intelligent Selection:
1. If `search_provider == "tavily"` and has key → Tavily.
2. If `prioritize_free_search == true` or no key → DuckDuckGo.
3. If Tavily fails → Automatic fallback to DuckDuckGo.

### 4.16 projects.rs — Multi-Project Management

#### Features:

- **Active project**: Resolved from `workspace.json.lastProject`.
- **History**: Maximum 15 projects, stored in `settings.json.project_history`.
- **Selection**: Normalizes paths (backslash → forward slash), deduplication, move-to-front.
- **Creation**: Creates directory + selects automatically.
- **Dynamic watcher**: Restarts on project switch.

### 4.17 filesystem.rs — Filesystem Operations

APIs exposed for the file explorer:

- `GET /api/v1/system/file-tree` — Complete active project tree.
- `POST /api/v1/system/read-file` — Read file contents.
- `POST /api/v1/system/save-file` — Save file contents.
- `POST /api/v1/system/create-item` — Create file or directory.
- `POST /api/v1/system/delete-item` — Delete file or directory.
- `POST /api/v1/system/rename-item` — Rename file or directory.
- `GET /api/v1/proxy/check-url` — Proxy for URL verification (Live Preview).

### 4.18 tasks.rs — Kanban System

- Storage in `.freyabot/team/tasks.json`.
- **States**: `PENDING` → `PLANNING` → `REVIEWING_PLAN` → `BUILDING` → `REVIEW` → `DONE` / `FAILED`.
- **SSE Stream**: `GET /api/v1/tasks/stream` for real-time updates.
- Full CRUD + streaming.

### 4.19 models.rs — Data Models

Defines the Rust structures for:

- `Task`, `TaskStatus`, `TaskStatusUpdate` — Kanban system.
- `LlmModelItem`, `SettingsPayload` — Configuration.
- `WorkspacePayload` — Workspace state (panels, tabs, etc.).
- `LiveMetrics` — Hardware metrics.

---

## 5. Frontend GUI in Detail

### 5.1 Main Components

#### `page.tsx` (51KB) — Main Dashboard:
- Chat with real-time SSE streaming.
- Markdown rendering with syntax-highlighted code.
- Mode buttons (PLAN/BUILD), effort selector, model selector.
- Agent status indicators.
- Persistent message history.

#### `FileExplorer.tsx` (44KB):
- Virtualized file tree.
- Context menu (create, rename, delete, copy, paste).
- Virtual clipboard (cut/copy/paste).
- Drag & Drop.
- File type icons.
- Integration with Watcher for real-time updates.

#### `MainWorkspace.tsx` (22KB):
- Tab system with Monaco Editor.
- Multi-file support.
- Automatic language detection.
- Saving via REST API.

#### `SettingsModal.tsx` (30KB):
- API key configuration (Gemini, Groq, SambaNova, OpenCode, Tavily).
- Enabled model selection.
- Customizable LLM endpoint.
- Web search configuration.
- Setup Wizard for first run.

#### `LivePreview.tsx` (27KB):
- Live web application preview.
- Embedded webview (Electron) or iframe.
- Automatic dev server port detection.
- Integrated navigation bar.

#### `SwarmDashboard.tsx` (13KB):
- Worker state visualization.
- Real-time activity via telemetry.
- Each Worker's status (idle/active/terminated).

#### `MetricsPanel.tsx` (11KB):
- CPU, RAM, GPU, VRAM gauges.
- Mission counters, success rate, savings.
- Updates every 5 seconds via SSE.

#### `BottomPanels.tsx` (6.5KB):
- Tabs: Terminal | System Logs.
- Integration with xterm.js for native terminal.
- Real-time system logs.

#### `XTermTerminal.tsx` (5.5KB):
- Native PTY terminal via node-pty.
- IPC communication with Electron.
- Adaptive resize.

#### `ProjectLauncher.tsx` (17KB):
- Existing project selector.
- New project creation.
- Native folder selection dialog.

### 5.2 Routing and Layout System

- `app/layout.tsx` — Root layout with Outfit + JetBrains Mono typography.
- `app/page.tsx` — Main dashboard (single functional route).
- `app/login/` — Login page.
- `app/editor-standalone/` — Standalone Monaco editor (popup).
- `app/preview-standalone/` — Standalone preview (popup).

### 5.3 Electron Shell (main.js)

#### Boot Sequence:

```
[1] Synchronous app:// scheme registration (before ready)
    ↓
[2] app.whenReady()
    ↓
[3] Protocol handler registration (app://)
    - Serves static files from gui/out/
    - SPA fallback → index.html
    ↓
[4] findFreePort(45000) — finds available port
    ↓
[5] spawnRustSidecar(port, userData)
    - Dev: freyabot-core/target/debug/freyabot-core.exe
    - Prod: resources/bin/freyabot-core.exe
    - Args: --port {port} --data-dir {userData}
    ↓
[6] waitForRust() — polling GET /health (max 20 attempts, 500ms)
    ↓
[7] Create BrowserWindow (1400x900, maximized)
    - Dev: http://localhost:3000
    - Prod: app://-/index.html?apiPort={port}
```

#### Multi-Terminal PTY Manager:

- Uses `node-pty` (native) for real terminals.
- Multi-tab support: each tab has its own PTY process.
- IPC Events: `terminal.create`, `terminal.input`, `terminal.kill`, `terminal.resize`.
- Automatic shell: `%COMSPEC%` or `powershell.exe`.
- Graceful fallback if node-pty is unavailable.

#### Graceful Shutdown:

1. User closes the window → `mainWindow.on('close')`.
2. `app-closing` is sent to the renderer.
3. The renderer has 4s for cleanup.
4. The window is destroyed and the Rust sidecar is killed.

---

## 6. System Protocols

### 6.1 Sovereign Protocol

The system registers itself as "alive" by writing its URL to a sentinel file:

```
{project_root}/.freyabot/api.url → "http://127.0.0.1:45000"
```

On shutdown (Ctrl+C or Graceful Shutdown), the sentinel is deleted. This allows:
- Detecting if the backend is active without HTTP polling.
- Multiple instances with different ports.

### 6.2 Agentic Loop

```
┌──────────────────────────────────────────────────┐
│                   AGENTIC LOOP                   │
│                                                  │
│  ┌─────┐     ┌──────────┐     ┌──────────────┐  │
│  │Step │────→│ LLM Call │────→│ Parse        │  │
│  │ N   │     │ (Stream) │     │ Response     │  │
│  └─────┘     └──────────┘     └──────┬───────┘  │
│                                       │          │
│                              ┌────────┴────────┐│
│                              │                  ││
│                        Tool Calls?        Text? ││
│                              │                  ││
│                    ┌─────────▼─────────┐       ││
│                    │ Execute Tools     │  STOP  ││
│                    │ (parallel)        │  ──→ 🏁││
│                    └─────────┬─────────┘       ││
│                              │                  ││
│                    ┌─────────▼─────────┐       ││
│                    │ Inject Results    │       ││
│                    │ as Tool Messages  │       ││
│                    └─────────┬─────────┘       ││
│                              │                  ││
│                    ┌─────────▼─────────┐       ││
│                    │ Step N+1          │       ││
│                    │ (loop continues)  │       ││
│                    └───────────────────┘       ││
│                                                  │
│  Stop Conditions:                                │
│  • step >= max_steps                             │
│  • Response without tool_calls (plain text)      │
│  • [STOP_AGENT] sentinel in tool result          │
│  • Error 429 (rate limit)                        │
│  • modifying_actions >= max_actions              │
└──────────────────────────────────────────────────┘
```

### 6.3 Anti-Amputation Protocol

When `finish_reason == "length"`:

```
[1] Save text generated so far
    ↓
[2] Inject message: "Continue from where you left off.
    Do NOT repeat anything. Start from the exact next character."
    ↓
[3] continuation_count++
    ↓
[4] If continuation_count < 5 → Repeat call
    If continuation_count >= 5 → Abort with warning
```

### 6.4 Auto-Healing (Self-Healing Loop)

##### Pre-Boot (self_healing.rs):
`BUILDING` tasks → Git verification → `DONE` or `REVIEW`.

##### Post-CLI (cli_bridge.rs, CLI mode):
```
[1] CLI finishes executing
    ↓
[2] validate_project(root, telemetry_tx)
    ↓
[3] If success → Break (success)
[3] If fail && attempt < 3:
    - Truncate stderr to 100 lines (Smart Truncate)
    - Re-inject error as context to the CLI
    - Retry
[3] If fail && attempt >= 3 → Final error
```

### 6.5 Agentic Shielding (Tail Appendment)

Directives injected at the end of the user prompt based on context:

- **`architect_prd`**: "MUST generate 3 markdown files using write_file + create tasks"
- **`feature_extension`**: "MUST generate feature-plan.md analyzing impact"
- **Build Mode**: "FORBIDDEN from writing code. DELEGATE via spawn_worker + dispatch_worker"
- **Execute Phase**: "MUST invoke execute_command. Do not respond with text only"

### 6.6 Deterministic State Machine

In Architect mode with Auto-Healing enabled:

```
State 1: No Master Task → CREATE master task
    ↓
State 2: mvp/prd.md doesn't exist → CREATE prd.md
    ↓
State 3: mvp/prd-technical.md doesn't exist → CREATE prd-technical.md
    ↓
State 4: mvp/budget.md doesn't exist → CREATE budget.md
    ↓
State 5: Only 1 task (Master) → BULK_CREATE phases + MARK master as DONE
    ↓
State 6: Everything ready → Free operation
```

---

## 7. Complete REST API

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/system/health` | Simple health check |
| `GET` | `/api/v1/health` | JSON health check |
| **Tasks** | | |
| `GET` | `/api/v1/tasks` | List tasks |
| `PATCH` | `/api/v1/tasks/:id/status` | Update status |
| `POST` | `/api/v1/tasks/create` | Create task |
| `GET` | `/api/v1/tasks/stream` | SSE task stream |
| **Filesystem** | | |
| `GET` | `/api/v1/system/file-tree` | File tree |
| `POST` | `/api/v1/system/read-file` | Read file |
| `POST` | `/api/v1/system/save-file` | Save file |
| `POST` | `/api/v1/system/create-item` | Create item |
| `POST` | `/api/v1/system/delete-item` | Delete item |
| `POST` | `/api/v1/system/rename-item` | Rename item |
| **Clipboard** | | |
| `GET` | `/api/v1/system/clipboard` | Clipboard state |
| `POST` | `/api/v1/system/clipboard/copy` | Copy |
| `POST` | `/api/v1/system/clipboard/cut` | Cut |
| `POST` | `/api/v1/system/clipboard/paste` | Paste |
| **Proxy** | | |
| `GET` | `/api/v1/proxy/check-url` | Verify URL |
| **Projects** | | |
| `GET` | `/api/v1/system/projects` | List projects |
| `POST` | `/api/v1/system/select-project` | Select project |
| `POST` | `/api/v1/system/projects/create` | Create project |
| **Knowledge** | | |
| `POST` | `/api/v1/system/assimilate` | Assimilate rule |
| **Settings** | | |
| `GET` | `/api/v1/system/settings` | Get configuration |
| `POST` | `/api/v1/system/settings` | Save configuration |
| `GET` | `/api/v1/system/workspace` | Get workspace |
| `POST` | `/api/v1/system/workspace` | Save workspace |
| **Session** | | |
| `GET` | `/api/v1/system/session` | Get session |
| `POST` | `/api/v1/system/session` | Save session |
| `DELETE` | `/api/v1/system/session` | Clear session |
| **Telemetry** | | |
| `GET` | `/api/v1/system/metrics/live` | Live metrics |
| `GET` | `/api/v1/system/telemetry` | SSE telemetry stream |
| **Webhooks** | | |
| `POST` | `/api/v1/webhooks/error-tracker` | Sentry webhook |
| **Orchestrator** | | |
| `POST` | `/api/v1/orchestrate/ask` | Send prompt to LLM (SSE) |
| `POST` | `/api/v1/orchestrate/resume` | Resume session |
| `POST` | `/api/v1/orchestrate/confirm` | Confirm turn |
| `POST` | `/api/v1/orchestrate/rollback` | Rollback changes |
| `GET` | `/api/v1/orchestrate/sessions` | List sessions |
| `GET` | `/api/v1/context/:role` | Get system prompt |
| **Engine** | | |
| `GET` | `/api/v1/system/engine/status` | LLM engine status |
| `POST` | `/api/v1/system/engine/install` | Install engine |
| `GET` | `/api/v1/system/engine/progress` | Download progress |

---

## 8. LLM Provider System

### Provider Hub (Multi-Agentic Hub):

```
┌─────────────────────────────────────────────────────────────┐
│                    PROVIDER DISPATCHER                       │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ LM Studio│  │ Google   │  │ Groq     │  │ SambaNova│   │
│  │ (Local)  │  │ Gemini   │  │ Cloud    │  │ Cloud    │   │
│  │ :1234/v1 │  │ /v1beta/ │  │ /openai/ │  │ /v1      │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           OpenCode Zen (big-pickle, MiniMax,         │   │
│  │           MiMo, Nemotron, free models)               │   │
│  │           opencode.ai/zen/v1                         │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Model Mapping:

| Model | Provider | Endpoint |
|-------|----------|----------|
| `freia-local` / any other | LM Studio | `http://127.0.0.1:1234/v1` |
| `gemini-2.5-flash`, `gemini-1.5-flash` | Google Gemini | `generativelanguage.googleapis.com` |
| `llama-3.3-70b-versatile`, `qwen3-32b`, `llama-4-scout-*` | Groq | `api.groq.com/openai/v1` |
| `Meta-Llama-3.3-70B-Instruct` | SambaNova | `api.sambanova.ai/v1` |
| `big-pickle`, `*minimax*`, `*-free`, `*mimo*`, `*nemotron*` | OpenCode Zen | `opencode.ai/zen/v1` |

### API Key Configuration (settings.json):

```json
{
  "apiKeys": {
    "gemini": "AIzaSy...",
    "groq": "gsk_...",
    "sambanova": "...",
    "opencode": "..."
  },
  "llm_endpoint": "http://127.0.0.1:1234/v1",
  "llm_api_key": "",
  "llm_max_tokens": -1,
  "llm_engine_mode": "api"
}
```

---

## 9. Project Directory Structure

```
freyabotos/
├── freyabot-core/                # Rust Backend (Axum)
│   ├── src/                      # Rust source code
│   ├── Cargo.toml                # Manifest and dependencies
│   └── target/                   # Compiled binaries
├── gui/                          # Next.js + Electron Frontend
│   ├── src/                      # TypeScript/React source code
│   ├── main.js                   # Electron main process
│   ├── preload.js                # Preload script (IPC)
│   ├── package.json              # Dependencies and scripts
│   ├── out/                      # Next.js static build
│   └── dist/                     # Electron build (NSIS)
├── ghost-bridge/                 # Auxiliary CLI Bridge (Rust)
│   └── src/                      # main.rs + lib.rs
├── scripts/                      # Auxiliary scripts
│   ├── mcp_server.py             # Experimental MCP server
│   └── validate_build.py         # Build validator
├── .freyabot/                    # System metadata
│   └── team/
│       ├── tasks.json            # Project Kanban
│       ├── sops/                 # Standard Operating Procedures
│       │   ├── architect_prd.md
│       │   └── feature_extension.md
│       ├── CODING_STANDARDS.md   # Coding rules
│       └── logs/                 # Terminal logs
├── mvp/                          # Architect-generated documents
│   ├── prd.md                    # Functional PRD
│   ├── prd-technical.md          # Technical PRD
│   └── budget.md                 # Budget and Roadmap
├── settings.json                 # Global configuration (legacy)
├── workspace.json                # Workspace state (legacy)
├── opencode.json                 # OpenCode CLI configuration
└── .env.local                    # Environment variables
```

### Persistent Data (outside the project):

```
%LOCALAPPDATA%/freyabot/          # (or .freyabot/data/ in portable mode)
├── settings.json                 # Global configuration
├── workspace.json                # Workspace state
├── session.json                  # Auth session
├── state/                        # Interrupted sessions
│   └── {session_id}.json
└── brain/
    └── micelio.json              # Vector database
```

---

## 10. Internal Conventions and Standards

### Ticket Identifiers:

Format: `FREIA-{number}` (e.g.: FREIA-74, FREIA-85, FREIA-100).

### Protocol Names:

| Name | Description |
|------|-------------|
| Agentic Shielding | Directive injection at the end of the prompt |
| Anti-Amputation Protocol | Automatic continuation when the LLM cuts due to token limit |
| Deterministic State Machine | Forced bootstrapping sequence |
| Build Enforcement | Direct coding prohibition in Build mode |
| Smart Truncate | Intelligent truncation of long outputs |
| Sovereign Bunker | Autonomy and portability concept |
| Mycelial Network (Mycelium) | Episodic memory based on embeddings |
| Sentinel | File Watcher + liveness detection file |
| VRAM Auto-Hemorrhage | Automatic cleanup of inactive Workers |
| Iron Coordinator | Swarm coordination system |

### Code Conventions:

- **Rust**: Edition 2021, async/await with Tokio, serde for serialization.
- **Frontend**: Functional React with hooks, strict TypeScript.
- **Paths**: Always relative to workspace. Absolute paths are forbidden for the LLM.
- **Logs**: Emojis as prefixes for visual categorization (🧠 = AI, 🛡️ = Security, 📡 = Network, etc.).
- **Persistence**: Atomic writing (tmp → rename) on all JSON files.

---

## 11. End-to-End Data Flow

### Example: User sends "Create a Button component"

```
[1] Frontend: POST /api/v1/orchestrate/ask
    {message: "Create a Button component", role: "forge", ui_mode: "build", ui_effort: "med"}
        │
[2] orchestrator.rs:
    ├── Workspace Awareness: Non-empty project → "forge" mode
    ├── context_builder: build_system_prompt()
    │   ├── Detects language: React/TSX
    │   ├── Reads active file (if any)
    │   ├── Gets Git state
    │   ├── Searches episodic memory (Mycelium)
    │   ├── Loads CODING_STANDARDS.md
    │   └── Generates PromptTicket with XML tags
    ├── Creates SessionSnapshot (pre-flight)
    └── Dispatches ask_opencode_cli_stream()
        │
[3] cli_bridge.rs (API Mode):
    ├── Determines provider (e.g.: Groq)
    ├── Configures endpoint + API key
    ├── Builds messages[] with system + user
    ├── Injects tools (get_tool_schemas)
    └── ENTERS THE AGENTIC LOOP
        │
    [Iteration 1]:
    ├── POST /chat/completions (streaming)
    ├── LLM responds: tool_call "write_file" {path: "src/Button.tsx", content: "..."}
    ├── Execute write_file() → File created
    ├── Inject result as message role: "tool"
    │
    [Iteration 2]:
    ├── POST /chat/completions (streaming)
    ├── LLM responds: plain text "I've created the component..."
    └── STOP (no more tool calls)
        │
[4] SSE Events emitted to frontend:
    ├── {type: "session_init", session_id: "..."}
    ├── {type: "status_update", message: "[Iteration 1/25]"}
    ├── {type: "status_update", message: "Tool: write_file → src/Button.tsx"}
    ├── {type: "action_completed", files: ["src/Button.tsx"]}
    ├── {type: "chunk", text: "I've created the component..."}
    └── "[DONE]"
        │
[5] Frontend:
    ├── Renders chunks in chat (streaming)
    ├── Updates File Explorer (fs_change)
    ├── Marks file as modified
    └── Enables Confirm/Rollback buttons
```

---

## 12. Persistence System

### Persistence Layers:

| Layer | Path | Content | Scope |
|-------|------|---------|-------|
| **Global** | `%LOCALAPPDATA%/freyabot/` | Settings, workspace, sessions | System |
| **Project** | `{project}/.freyabot/` | Tasks, SOPs, logs, standards | Project |
| **Memory** | `{project}/.freia/memory/` | FREIA.md, session.json | Project |
| **Mycelium** | `{data_dir}/brain/micelio.json` | Embeddings + memories | System |
| **Sessions** | `{data_dir}/state/` | Session snapshots | Temporary |

### Automatic Migrations:

1. **Roaming → LocalAppData**: Migrates data from `%APPDATA%/freyabot/` to `%LOCALAPPDATA%/freyabot/`.
2. **Root → Data Dir**: Migrates files from the project root to the data dir.
3. **Auto-cleanup**: Sessions older than 24h are automatically deleted.

---

## 13. Security and Sandboxing

### Command Sandboxing:

- **Strict whitelist**: Only `npm`, `npx`, `bun`, `cargo`, `git`, `rustc`, `node`.
- **Timeout**: 60s (normal) / 180s (install/build).
- **Kill on drop**: Orphaned processes are automatically killed.

### Path Sanitization:

- All LLM paths are resolved relative to the active project.
- Path traversal blocked: `if !full_path.starts_with(&root)`.
- Leading slashes automatically removed.

### CORS:

- Allowed origins: `app://-`, `http://localhost:3000`, `http://127.0.0.1:3000`.
- Credentials allowed.
- Explicit methods: GET, POST, PUT, DELETE, PATCH, OPTIONS.

### Electron:

- `nodeIntegration: false` — No direct Node access from the renderer.
- `contextIsolation: true` — IPC via preload.js.
- `webviewTag: true` — Only for Live Preview.

### Rate Limiting:

- **Inter-iteration delay**: 3 seconds between agentic loop iterations.
- **429 Detection**: Immediate stop with descriptive message to the user.

---

> **Freyabot OS** — AI agent orchestration system built as a sovereign and portable workstation.
