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
Routes commands to sub-agents and enforces workflow order and constraints.

### Project_Initializer
Runs `/init` workflow: generates project `AGENTS.md`, creates `tasks.yaml` per schema, writes `.gitignore`, initializes git, runs `/bd-init` (creates `.beads/`), and creates `./plans/`.

### Task_Manager
Owns `./tasks.yaml` only: creates tasks from `/add-task` input (`$ARGUMENTS`), updates status transitions, updates notes/metadata, and manages Beads mirroring via beads subagent.

### plan (default agent)
Creates implementation plan for a task and saves it to `./plans/plan-<task-id>.md`. It is analysis-first and must not implement code changes.

### build (built-in agent)
Implements the plan from `./plans/plan-<task-id>.md`. It can modify code, but must respect `dont_touch_paths` and `delegable`.

### Code_Reviewer
Manual step invoked by user via `/review`. Reviews the session changes (AI + human) with a checklist and issues APPROVE/REJECT, separating AI blockers vs user warnings.

### Repo_Manager
Manual step invoked by user via `/commit`. Inspects changes and generates copy-paste Git + SVN commands and 2–3 Conventional Commit messages.

---

## Workflow steps (complete cycle)

### Step 1: Initialize project
Command: `/init` → Agent: `Project_Initializer`

Creates/ensures:
- `AGENTS.md` (project-local conventions)
- `tasks.yaml` (canonical schema)
- `.gitignore`
- git repo initialized (required prerequisite for Beads init)
- Initialize Beads via OpenCode command `beads:init` (creates `.beads/` directory for persistence). Fallback to bash `bd init` if the command is unavailable.
- Ensure `.beads/` is ignored in `.gitignore`
- `./plans/` directory

Notes:
- `/bd-init` must run AFTER `git init` completes.
- Beads initialization is part of `/init` to enable recovery from the beginning. Use `beads:init` command first, fallback to `bd init` via bash if needed.


### Step 2: Create task
Command: `/add-task <input>` → Agent: `Task_Manager`

Rules:
- Input source is `$ARGUMENTS`, not conversation context.
- ID format: `T-NNNN`, next ID is always `max + 1`.
- Defaults: `status: TODO`, `delegable: true`, `priority: 3`, beads fields null with `mirrored: false`.

### Step 3: Plan
Command: `/plan [T-NNNN]` → Agent: `plan`

Output:
- Writes `./plans/plan-<task-id>.md`.
- When planning starts, Task_Manager should set task `TODO → DOING` and mirror to Beads (AI-only).

### Step 4: Build
Command: `/build [T-NNNN]` → Agent: `build`

Rules:
- Implement plan sequentially; do not skip items.
- Respect `dont_touch_paths` (stop and ask if required).
- Respect `delegable: false` (do not implement).

### Step 5: Review (manual)
Command: `/review` → Agent: `Code_Reviewer`

Rules:
- Review unstaged changes first; if none, review staged changes.
- Always output APPROVE or REJECT.
- If REJECT:
  - AI-generated blockers require user choice (re-implement / manual fix / override).
  - User-written issues are warnings (non-blocking).

### Step 6: Task state update
Agent: `Task_Manager`

Rules:
- If approved and task is AI-related: transition `DOING → DONE` and close Beads issue (keep metadata as history).
- If blocked due to AI-generated issues: transition `DOING → BLOCKED`, append issues to notes, update Beads issue.
- If user skips: set `SKIP` with reason and close Beads issue if mirrored.

### Step 7: Version control (manual)
Command: `/commit` → Agent: `Repo_Manager`

Rules:
- Always output Git + SVN command blocks unless user explicitly disables one (“no SVN” / “no Git”).
- Ask user to stage all or select files (interactive choice).
- Provide 2–3 Conventional Commit messages; include `Refs: T-NNNN` only if there is an active DOING task.

---

## Beads integration (AI-only)

Purpose:
- Recovery for in-flight AI tasks (`DOING`, `BLOCKED`) across disconnects/compaction.

Rules:
- Source of truth remains `tasks.yaml`.
- Mirror only `DOING` / `BLOCKED`.
- Close Beads issue when task becomes `DONE` or `SKIP` and keep metadata for history.
- Beads operations are done via the OpenCode command `beads:init` for initialization. Use `bd` CLI via bash only as explicit fallback when the OpenCode command is unavailable.

---

## Safety boundaries

1) `dont_touch_paths`
- Agents must not propose patches or edits in restricted paths; must ask for approval.

2) `delegable`
- If `delegable: false`, AI agents may only propose/document; no implementation.

3) Sensitive data
- Do not persist secrets or client-sensitive information in tasks, Beads, or long-term notes.
