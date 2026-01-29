# Task System (tasks.yaml + AI-only Beads)

## Purpose
This document defines the authoritative task system used by OpenCode agents.
It standardizes ./tasks.yaml, task lifecycle rules, and the AI-only Beads integration for persistence and incident recovery.

## Files and locations
- The tasks file MUST exist at the repository root: ./tasks.yaml.
- tasks.yaml is the single source of truth for all tasks.

## Agents affected
Primary:
- **Task_Manager**: creates/updates tasks, enforces schema, keeps tasks.yaml authoritative, manages Beads metadata.

Secondary:
- **plan**: reads tasks.yaml to plan work and propose next steps.
- **Repo_Manager**: uses tasks.yaml to propose commit messages and copy-paste command sequences.
- **Code_Reviewer**: uses tasks.yaml as acceptance context while reviewing changes.
- **Command_Runner**: executes commands only when explicitly asked; may reference task metadata (paths, dont_touch_paths).

---

## Source of truth
- tasks.yaml is authoritative for:
  - Task existence
  - Task status (TODO/DOING/DONE/BLOCKED/SKIP)
  - Notes and constraints
  - AI delegation flags
  - Path hints and safety boundaries
  - Beads metadata stored inside each task

Beads is **non-authoritative** and is used only by the AI to persist in-flight execution context and recover after incidents.

---

## Canonical YAML structure
tasks.yaml MUST follow this structure:

```yaml
tasks:
  - id: T-0001
    title: "Short description of the task"
    status: TODO        # TODO | DOING | DONE | BLOCKED | SKIP
    delegable: true       # true if this task can be delegated to the AI
    priority: 1           # integer (1 = highest priority, 5 = lowest)
    paths_hint:
      - "src/main/java/..."
      - "src/test/java/..."
    dont_touch_paths:
      - "db/migrations/"
    notes: |
      Free-form text for additional context
      (bug reproduction steps, business notes, links, extra constraints)
    beads:                # AI-managed metadata for task persistence and recovery
      bead_id: null       # Beads issue ID linked to this task (string when mirrored, else null)
      mirrored: false     # true when task is mirrored in Beads (DOING/BLOCKED status)
      last_sync_at: null  # ISO timestamp of last beads sync
      last_error: null    # Last error encountered during beads sync (if any)
      session_id: null    # Session ID currently working on this task
      agent: null         # Agent name that last worked on this task
```

## Field definitions (per task)
- **id** (required): stable task identifier in format T-NNNN (e.g., T-0001, T-0002), MUST NOT change.
- **title** (required): short, imperative, unambiguous.
- **status** (required): TODO | DOING | DONE | BLOCKED | SKIP.
- **delegable** (required): whether AI agents are allowed to execute this task.
- **priority** (required): integer where 1 is highest priority, 5 is lowest.
- **paths_hint** (optional): list of likely relevant paths; helps reduce search ambiguity.
- **dont_touch_paths** (optional): hard safety boundary; agents MUST NOT edit files under these paths.
- **notes** (optional): free-form context; keep it factual and decision-oriented.
- **beads** (required): AI-managed metadata object for task persistence; see field definitions below.

## Beads field definitions (per task)
- **bead_id**: Beads issue identifier when task is mirrored (string), else null.
- **mirrored**: Boolean flag indicating if task is currently mirrored in Beads system.
- **last_sync_at**: ISO 8601 timestamp of last beads synchronization.
- **last_error**: Error message from last sync attempt (null if no errors).
- **session_id**: Current session identifier working on this task.
- **agent**: Name of the agent that last modified this task.

All beads fields default to null (or false for mirrored) when task is created. When task status becomes DONE or SKIP, beads metadata is left as-is for historical tracking.

## Status semantics
- **TODO**: Not started.
- **DOING**: Actively in progress (there should be an immediate next action).
- **BLOCKED**: Cannot proceed without dependency/input.
- **DONE**: Completed and validated (according to notes/acceptance).
- **SKIP**: Intentionally not doing it (must be justified in notes).

## Delegation rules (delegable)
If **delegable: false**, agents MUST NOT implement the task (no code changes, no commands) and should only:
- ask clarifying questions,
- propose options,
- document what a human should do.

If **delegable: true**, agents may proceed normally, while still respecting permissions/tools boundaries.

## Path safety rules
- **paths_hint** is guidance (helps focus).
- **dont_touch_paths** is a constraint:
  - Agents MUST NOT edit, write, or generate patches affecting those paths.
  - If a change is required in dont_touch_paths, the agent must stop and ask for explicit approval and a process (usually human-handled).

---

## Beads integration (AI-only persistence)
Beads must be used exclusively by the AI to provide persistence during execution and for incident recovery.

### 1) What Beads is for
- Persist in-flight context for tasks in DOING or BLOCKED.
- Recover after incidents (crash, session loss, compaction loss).

### 2) What Beads is NOT for
- It is not the primary task manager.
- Humans should not operate Beads directly.
- Task status must never be driven by Beads; tasks.yaml remains authoritative.

### 3) Mirroring policy
- Mirror ONLY tasks with status: DOING or status: BLOCKED.
- Do NOT mirror tasks with status: TODO, DONE, or SKIP.
- Update task.beads.mirrored accordingly.

### 4) What can be stored in external Beads issues
**Allowed** (recovery-oriented only):
- Immediate next action
- Copy-paste commands to run (do not execute)
- Links: files, commits, PRs, issues
- Blockers and required inputs

**Forbidden**:
- Secrets/tokens/credentials
- Client proprietary or sensitive operational data

---

## Task_Manager responsibilities (mandatory)
**Task_Manager MUST:**
- Enforce schema and keep tasks.yaml consistent.
- Ensure every task has a stable task id in format T-NNNN with zero-padding.
- Next ID calculation: parse existing IDs, find max number, increment by 1, format as T-NNNN.
- When a task becomes DOING or BLOCKED:
  - Create or update the external Beads issue (AI-only),
  - Update task.beads fields: bead_id, mirrored (true), last_sync_at, session_id, agent.
- When a task becomes DONE or SKIP:
  - Update status in tasks.yaml.
  - Leave task.beads fields as-is for historical tracking.
  - Optionally leave the external Beads issue as historical context.

**Task_Manager MUST NOT:**
- Change task status based on Beads alone.
- Store sensitive info in Beads or in tasks.yaml.
- Edit anything under dont_touch_paths.

---

## Incident recovery procedure
When recovering from an incident:
1. Read ./tasks.yaml first (authoritative).
2. Identify tasks in DOING or BLOCKED status.
3. For each in-flight task:
   - Check task.beads.bead_id to find linked Beads issue.
   - Recover missing execution context from external Beads issue.
   - Write recovered context into task.notes or other task fields.
4. Continue work with tasks.yaml as authoritative.

## Conflict resolution
If external Beads and tasks.yaml disagree:
- **tasks.yaml wins** for status and task definition.
- Use Beads only to reconstruct missing context, then write it back to tasks.yaml.