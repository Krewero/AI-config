# OpenCode Workflow (Command-driven)

## Purpose
This document defines the authoritative workflow for OpenCode in this environment: command routing, agent responsibilities, handoffs, and safety boundaries.

## Non-negotiable principles
1. `tasks.yaml` is the single source of truth for AI task state.
2. Workflow is command-driven: `/init`, `/add-task`, `/plan`, `/build`, `/review`, `/commit`.
3. MCP-first: prefer Git MCP tools over bash for inspection and metadata.
4. Copy-paste-first for VCS: Repo_Manager generates commands; the user executes them.
5. Safety boundaries are mandatory: respect `dont_touch_paths` and `delegable` in tasks.
6. No secrets in persistence: never store credentials/tokens in tasks or Beads.

---

## Agents and roles

### OCA (Orchestrator)
Primary agent. Routes commands to sub-agents, validates prerequisites, and orchestrates multi-step workflows (`/plan`, `/review`). Does NOT implement code or modify tasks.yaml directly.

### Project_Initializer
Runs `/init` workflow: generates project `AGENTS.md`, creates `tasks.yaml` per schema, writes `.gitignore`, initializes git, runs `beads:init` (creates `.beads/`), and creates `./plans/`.

### Task_Manager
Owns `./tasks.yaml` only: creates tasks from `/add-task` input (`$ARGUMENTS`), updates status transitions (TODO→DOING→DONE/BLOCKED), updates notes/metadata, and manages Beads mirroring via @beads-task-agent.

### plan
Creates implementation plan for a task in DOING status and saves it to `./plans/plan-<task-id>.md`. Analysis-first, must not implement code changes. Does NOT update task status (Task_Manager already did).

### build (built-in agent)
Implements the plan from `./plans/plan-<task-id>.md`. Can modify code, but must respect `dont_touch_paths` and `delegable`. Does NOT update task status.

### Code_Reviewer
Reviews code changes with checklist, issues APPROVE/REJECT decision, separates AI blockers vs user warnings. Read-only, does NOT modify code or update task status.

### Repo_Manager
Inspects changes via MCP and generates copy-paste Git + SVN commands with Conventional Commit messages. NEVER executes commands automatically.

---

## Workflow steps (complete cycle)

### Step 1: Initialize project
Command: `/init`
Agent: `Project_Initializer`

Creates/ensures:
- `AGENTS.md` (project-local conventions)
- `tasks.yaml` (canonical schema)
- `.gitignore` (stack-appropriate)
- `.git/` (git repository initialized)
- `.beads/` (Beads persistence via `beads:init`)
- `./plans/` (plan storage directory)

Notes:
- `beads:init` must run AFTER `git init` completes.
- Fallback to bash `bd init` if OpenCode command unavailable.
- `.beads/` must be in `.gitignore`.

---

### Step 2: Create task
Command: `/add-task <input>`
Agent: `Task_Manager`

Rules:
- Input source is `$ARGUMENTS`, not conversation context.
- ID format: `T-NNNN`, next ID is always `max + 1`.
- Defaults: `status: TODO`, `delegable: true`, `priority: 3`, beads fields null with `mirrored: false`.

---

### Step 3: Plan (two-step orchestration)
Command: `/plan [T-NNNN]`
Orchestration: `OCA` → `Task_Manager` → `plan`

**Step 3a: OCA delegates to Task_Manager**
- Task_Manager parses input (task ID or auto-select)
- Validates task (exists, TODO, delegable)
- Updates status: `TODO → DOING`
- Mirrors to Beads (creates issue)
- Terminates

**Step 3b: OCA delegates to plan**
- plan finds task in DOING status
- Analyzes codebase
- Creates `./plans/plan-<task-id>.md`
- Terminates

Output:
- Task status: DOING
- Beads issue created
- Plan file ready for `/build`

---

### Step 4: Build
Command: `/build [T-NNNN]`
Orchestration: `OCA` (validation) → `build`

OCA validates:
- Task exists with status=DOING
- Plan file exists: `./plans/plan-<task-id>.md`

build implements:
- Follows plan checklist sequentially
- Respects `dont_touch_paths`
- Does NOT execute tests or builds
- Does NOT update task status

Output:
- Code changes implemented
- Task remains in DOING (awaiting review)

---

### Step 5: Review and Status Update (two-step orchestration)
Command: `/review`
Orchestration: `OCA` → `Code_Reviewer` → `Task_Manager`

**This is a combined step: review + status update happen together.**

**Step 5a: OCA delegates to Code_Reviewer**
- Analyzes uncommitted changes (unstaged first, staged fallback)
- Applies code-review-checklist
- Issues APPROVE or REJECT decision
- Identifies AI blockers vs user warnings
- Terminates with decision output

**Step 5b: OCA reads decision and delegates to Task_Manager**

OCA reads Code_Reviewer output and determines action:

| Decision | AI Blockers? | Action |
|----------|--------------|--------|
| APPROVE | - | DOING → DONE |
| REJECT | Yes | DOING → BLOCKED |
| REJECT | No (user warnings only) | DOING → DONE |

Task_Manager executes:
- Updates task status per OCA instruction
- Closes or updates Beads issue
- Terminates

Output:
- Task status updated (DONE or BLOCKED)
- Beads issue closed or updated
- Ready for `/commit` (if DONE)

---

### Step 6: Version control
Command: `/commit`
Agent: `Repo_Manager`

Rules:
- Always output BOTH Git + SVN command blocks (unless user disables one)
- Ask user: stage all or select files
- Provide 2–3 Conventional Commit messages
- Include `Refs: T-NNNN` only if changes match task's `paths_hint`
- NEVER execute commands automatically

Output:
- Copy-paste ready commands
- User executes manually

---

## Command → Agent routing summary

| Command | Agent(s) | Orchestration |
|---------|----------|---------------|
| `/init` | Project_Initializer | Single step |
| `/add-task` | Task_Manager | Single step |
| `/plan` | Task_Manager → plan | Two-step (OCA orchestrates) |
| `/build` | build | Single step (OCA validates) |
| `/review` | Code_Reviewer → Task_Manager | Two-step (OCA orchestrates) |
| `/commit` | Repo_Manager | Single step |

---

## Beads integration (AI-only)

Purpose:
- Recovery for in-flight AI tasks (`DOING`, `BLOCKED`) across disconnects/compaction.

Rules:
- Source of truth remains `tasks.yaml`.
- Mirror only `DOING` / `BLOCKED` status tasks.
- Close Beads issue when task becomes `DONE` or `SKIP`.
- Keep beads metadata in tasks.yaml for historical tracking.
- Beads operations delegated to @beads-task-agent (from opencode-beads plugin).

---

## Safety boundaries

### 1) `dont_touch_paths`
Agents must NOT propose patches or edits in restricted paths without explicit user approval.

### 2) `delegable`
If `delegable: false`, AI agents may only propose/document; no implementation allowed.

### 3) Sensitive data
Do NOT persist secrets, credentials, or client-sensitive information in:
- tasks.yaml
- Beads issues
- Long-term notes or memory

---

## Status transitions reference

| From | To | Trigger | Who |
|------|-----|---------|-----|
| TODO | DOING | `/plan` starts | Task_Manager |
| DOING | DONE | `/review` approves | Task_Manager |
| DOING | BLOCKED | `/review` rejects (AI blockers) | Task_Manager |
| BLOCKED | DOING | User resolves blocker | Task_Manager |
| Any | SKIP | User decides to skip | Task_Manager |

Final states (cannot reopen): `DONE`, `SKIP`

See `status-transition.md` for complete rules.