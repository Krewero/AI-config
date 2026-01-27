# Development Workflow

## Purpose
This document defines the complete development workflow used by OpenCode agents.
It describes how work flows through specialized agents, their roles and responsibilities, handoff rules, and the safety boundaries enforced throughout execution.

## Scope
This workflow applies to all repository work managed through OpenCode.
Every agent must follow the handoff rules and safety boundaries defined here.

---

## Core principles (non-negotiable)
1. **tasks.yaml is the source of truth** for all task state and progress.
2. **plan is the default agent** for analysis and planning (read-only, no code changes).
3. **build is the implementation agent** (built-in, full tools access).
4. **MCP tools are preferred** over bash for Git/GitHub operations when available.
5. **Copy-paste first for Git/SVN**: propose commands; do not execute destructive actions automatically.
6. **Safety boundaries are enforced**: `dont_touch_paths` and delegability must be respected.

---

## Agents (roles and responsibilities)

### Command_Runner
**Purpose**: Execute user-provided shell commands and initialization routines.

Responsibilities:
- Initialize project structure (e.g., `/init` for AGENTS.md, `/create-task-file` for tasks.yaml).
- Execute explicit shell commands requested by the user.
- Create basic project files when needed.

Constraints:
- Does not plan, write code, or perform analysis.
- Strictly limited to command execution and basic file creation.
- Does not have access to edit tools or skills.

---

### Task_Manager
**Purpose**: Authoritative owner of `./tasks.yaml`; creates, updates, and maintains task state and Beads metadata.

Responsibilities:
- Create new tasks from user prompts following the canonical YAML schema.
- Update task status throughout the workflow (TODO → DOING → DONE/BLOCKED/SKIP).
- Maintain Beads integration (AI-only) for task persistence:
  - Create/update Beads issues when task status becomes DOING or BLOCKED.
  - Update `beads.mapping.by_task_id` and per-task `beads: {}` metadata.
- Keep `tasks.yaml` consistent and deterministic.
- Use persistent memory (if available) to store stable project context (non-sensitive conventions, repo structure).
- Work in background during workflow steps 3-6 to keep task state synchronized.

Constraints:
- Operates only on `tasks.yaml` (does not edit other project files).
- MUST NOT change task status based on Beads alone; `tasks.yaml` is authoritative.
- MUST NOT store secrets or client-sensitive data in tasks or Beads.
- MUST respect `dont_touch_paths` (but `tasks.yaml` is always allowed).

---

### plan (default agent)
**Purpose**: Analysis and planning without making code changes.

Responsibilities:
- Read tasks from `tasks.yaml` created by Task_Manager.
- Analyze code, context, and requirements.
- Create implementation plans as internal notes/checklists (not as separate tasks in tasks.yaml).
- Suggest changes and propose next steps.
- Coordinate workflow but delegate actual implementation to build.

Constraints:
- Read-only mode: does not modify files.
- Does not create tasks in `tasks.yaml` (Task_Manager does).
- Plans are internal notes/checklists, not persisted as separate task entries.

---

### build (built-in agent)
**Purpose**: Default primary agent with all tools enabled for code implementation.

Responsibilities:
- Implement code changes following the plan created by plan.
- Modify files with new implementations.
- Generate diffs to show output to the user.
- Full access to file operations and system commands.

Constraints:
- MUST respect `dont_touch_paths`.
- MUST respect `delegable` flag (if `delegable: false`, do not implement).

---

### Code_Reviewer
**Purpose**: Review code changes and diffs against tasks.yaml and software engineering best practices.

Responsibilities:
- Review generated code changes (from build or user-written code).
- Apply systematic code review checklist.
- Evaluate code quality, scalability, and maintainability.
- Structure feedback as: Blockers / Warnings / Suggestions / Questions.
- Use `tasks.yaml` for acceptance context (Definition of Done).
- Approve solid implementations or reject with actionable feedback for re-planning.

Constraints:
- Read-only mode: does not modify files.
- If blockers are found, stop and report; do not proceed.

---

### Repo_Manager
**Purpose**: Generate copy-paste Git and SVN commands for versioning; never execute them.

Responsibilities:
- Inspect repository state (status, diff, log, branches) using MCP when available.
- Generate commit commands for both personal Git repos and company SVN repos.
- Suggest conventional commit messages based on tasks.yaml context.
- Provide copy-paste shell command sequences (ordered, deterministic).
- Link commits to tasks (e.g., `Refs: TASKS T-0004`).

Constraints:
- MUST NOT execute any write/push/merge/tag/commit commands automatically.
- MUST propose commands in a copy-pasteable block.
- Handles both Git and SVN (user uses Git for personal, SVN for company repos).

---

## Workflow steps (complete cycle)

### Step 1: Project initialization
**Agent**: Command_Runner

Actions:
- User invokes initialization commands (e.g., `/init`, `/create-task-file`).
- Command_Runner executes commands to create:
  - `AGENTS.md` (project rules).
  - `tasks.yaml` (task management file with canonical schema).
  - Other initialization files as needed.

Output:
- Project structure ready for task-based development.

---

### Step 2: Task creation
**Agent**: Task_Manager

Actions:
- User provides a prompt describing the task (e.g., "Create a task to fix logging in method X in class Y, distinguish debug vs production logs, especially in try-catch blocks").
- Task_Manager:
  - Interprets the prompt.
  - Creates a new task in `tasks.yaml` following the canonical YAML schema (id, title, status: TODO, delegable, priority, paths_hint, dont_touch_paths, notes).
  - Generates a stable `task_id` (e.g., `T-001`).

Output:
- New task in `tasks.yaml` with status `TODO`.
- Task is ready for planning and execution.

---

### Step 3: Planning and analysis
**Agent**: plan (default agent)

Actions:
- plan reads the task from `tasks.yaml`.
- plan analyzes:
  - Relevant code context.
  - Requirements and constraints.
  - Acceptance criteria.
- plan creates an implementation plan as internal notes/checklist (not persisted as separate tasks).
- plan proposes next steps.

**Agent**: Task_Manager (background)
- Task_Manager updates task status to `DOING`.
- Task_Manager creates/updates Beads issue (AI-only) and updates `beads.mapping.by_task_id`.

Output:
- Implementation plan ready.
- Task status: `DOING`.
- Beads issue created for persistence.

---

### Step 4: Code implementation
**Agent**: build (built-in)

Actions:
- build implements the code changes following the plan created by plan.
- build modifies files with new implementations.
- build generates diffs to show output to the user.

**Agent**: Task_Manager (background)
- Task_Manager monitors progress and updates task notes/metadata as needed.

Output:
- Code changes implemented.
- Diffs available for review.

---

### Step 5: Code review
**Agent**: Code_Reviewer + User

Actions:
- Code_Reviewer reviews the generated code changes and diffs.
- Code_Reviewer applies systematic checklist (correctness, security, maintainability, performance, observability, API contracts, config readiness).
- Code_Reviewer structures feedback: Blockers / Warnings / Suggestions / Questions.
- User reviews the code and provides feedback.

Output:
- Review feedback (approve or reject with actionable items).
- If blockers exist, stop and report to plan for re-planning.

---

### Step 6: Task completion
**Agent**: Task_Manager

Actions:
- Based on review outcome (from Code_Reviewer and user):
  - If approved: Task_Manager updates task status to `DONE`.
  - If rejected/blocked: Task_Manager updates status to `BLOCKED` or adds notes with required changes.
- Task_Manager updates final notes in `tasks.yaml`.
- Task_Manager does NOT maintain parallel Beads workflow (Beads is recovery-only; leave issue as historical context if needed).

Output:
- Task status updated to `DONE` (or `BLOCKED` with clear notes).
- Clean, auditable task history in `tasks.yaml`.

---

### Step 7: Version control (Git/SVN)
**Agent**: Repo_Manager

Actions:
- User asks to prepare commit/push.
- Repo_Manager:
  - Inspects repository state using MCP (git status, diff, log).
  - Reads `tasks.yaml` to understand completed work.
  - Generates 2–4 conventional commit message options.
  - Provides copy-paste command sequences for:
    - **Git** (personal repos): `git add`, `git commit`, `git push`.
    - **SVN** (company repos): `svn add`, `svn commit`.
  - Links commit to tasks (e.g., `Refs: TASKS T-0004`).

Output:
- Commit message options.
- Copy-paste command block for Git and/or SVN.
- User executes commands manually.

---

## Beads integration (AI-only persistence)

### Purpose
Beads is used exclusively by Task_Manager as a persistence layer for in-flight tasks (DOING/BLOCKED) to enable recovery after crashes, session loss, or compaction.

### Key rules
- **AI-only**: Humans do not interact with Beads directly.
- **Mirroring policy**: Mirror ONLY tasks with status `DOING` or `BLOCKED`.
- **Source of truth**: `tasks.yaml` is always authoritative; Beads is for recovery only.
- **Metadata**: Task_Manager maintains:
  - Top-level `beads.mapping.by_task_id` (canonical mapping: task_id → bead_id).
  - Per-task `beads: {}` section (bead_id, mirrored, last_sync_at, last_error).

### Recovery workflow
If an incident occurs:
1. Read `./tasks.yaml` first (authoritative).
2. Identify tasks in `DOING` or `BLOCKED` and their `task_id`.
3. For each in-flight task, use `beads.mapping.by_task_id` to locate Beads issue.
4. Recover missing execution context from Beads and write it into `tasks.yaml` notes.
5. Continue work with `tasks.yaml` as authoritative.

---

## Safety boundaries (enforced by all agents)

### 1) dont_touch_paths
- Defined per task in `tasks.yaml`.
- Agents MUST NOT edit, write, or generate patches for paths under `dont_touch_paths`.
- If a change is required, agent MUST stop and ask for explicit approval.

### 2) delegable flag
- If `delegable: false`, agents MUST NOT implement the task (no code changes, no commands).
- Only propose, document, or ask clarifying questions.

### 3) Secrets and sensitive data
- MUST NOT store secrets, tokens, credentials, or client-sensitive data in:
  - `tasks.yaml`
  - Beads issues
  - Persistent memory
- If sensitive data is provided, keep it out of persistence and suggest rotation/revocation.

---

## Summary (quick reference)
- **Initialization**: Command_Runner creates project structure.
- **Task creation**: Task_Manager creates tasks in `tasks.yaml` from user prompts.
- **Planning**: plan analyzes and creates implementation plans (internal notes/checklist).
- **Implementation**: build writes code following the plan.
- **Review**: Code_Reviewer + user approve or reject changes.
- **Completion**: Task_Manager updates task status (DONE/BLOCKED).
- **Versioning**: Repo_Manager generates copy-paste Git/SVN commands.
- **Persistence**: Task_Manager uses Beads (AI-only) for in-flight task recovery.
- **Safety**: `dont_touch_paths`, `delegable`, no secrets in persistence.
