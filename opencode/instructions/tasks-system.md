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
  - Beads metadata stored inside tasks.yaml

Beads is **non-authoritative** and is used only by the AI to persist in-flight execution context and recover after incidents.

---

## Canonical YAML structure
tasks.yaml MUST follow this structure:

```yaml
tasks:
  - id: "T-001"
    title: "Short description of the task"
    status: "TODO"        # TODO | DOING | DONE | BLOCKED | SKIP
    delegable: true       # true if this task can be delegated to the AI
    priority: 1           # optional integer (1 = highest priority)
    paths_hint:
      - "src/main/java/..."
      - "src/test/java/..."
    dont_touch_paths:
      - "db/migrations/"
    notes: |
      Free-form text for additional context
      (bug reproduction steps, business notes, links, extra constraints)
    beads: {}  # optional per-task beads metadata, may be omitted if not used yet

# Top-level Beads metadata (AI-only)
beads:
  enabled: true
  mode: "ai-only"
  mapping:
    by_task_id: {}  # task_id -> bead_id (string)
  sync:
    last_sync_at: null
    last_error: null
```

## Field definitions (per task)
- **id** (required): stable identifier (e.g., T-001), MUST NOT change.
- **title** (required): short, imperative, unambiguous.
- **status** (required): TODO | DOING | DONE | BLOCKED | SKIP.
- **delegable** (required): whether AI agents are allowed to execute this task.
- **priority** (optional): integer where 1 is highest priority.
- **paths_hint** (optional): list of likely relevant paths; helps reduce search ambiguity.
- **dont_touch_paths** (optional): hard safety boundary; agents MUST NOT edit files under these paths.
- **notes** (optional, recommended): free-form context; keep it factual and decision-oriented.
- **beads** (optional): per-task Beads metadata; see "Beads integration" section.

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

### 4) Required top-level Beads section (authoritative mapping)
The top-level **beads.mapping.by_task_id** is the canonical mapping:
- **key**: task_id (e.g., T-001)
- **value**: bead_id (string)

### 5) Per-task beads object (task-local metadata)
Each task may contain **beads: {}** to store task-specific metadata and avoid ambiguity.

Recommended keys inside task.beads:

```yaml
beads:
  bead_id: null           # string when created, else null
  mirrored: false         # true when DOING/BLOCKED is mirrored into Beads
  last_sync_at: null      # ISO timestamp
  last_error: null        # short, non-sensitive error
```

Rules:
- **task.beads.bead_id** MUST match **beads.mapping.by_task_id[task_id]** when present.
- If both exist and differ, treat tasks.yaml as corrupted state and stop for clarification.

### 6) What can be stored in Beads issues
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
- Ensure every task has a stable id.
- When a task becomes DOING or BLOCKED:
  - Create or update the Beads issue (AI-only),
  - Update BOTH:
    - beads.mapping.by_task_id[task_id],
    - task.beads fields (bead_id, mirrored, last_sync_at, last_error).
- When a task becomes DONE or SKIP:
  - Update tasks.yaml only.
  - Optionally leave the Beads issue as historical context, but do not maintain a parallel workflow.

**Task_Manager MUST NOT:**
- Change task status based on Beads alone.
- Store sensitive info in Beads or in tasks.yaml.
- Edit anything under dont_touch_paths.

---

## Incident recovery procedure
When recovering from an incident:
1. Read ./tasks.yaml first (authoritative).
2. Identify tasks in DOING or BLOCKED and their id.
3. For each in-flight task:
   - Resolve bead linkage via beads.mapping.by_task_id[id] (preferred),
   - Validate against task.beads.bead_id if present.
4. Recover missing execution context from Beads and write it into task.notes (or other task fields).
5. Continue work with tasks.yaml as authoritative.

## Conflict resolution
If Beads and tasks.yaml disagree:
- **tasks.yaml wins** for status and task definition.
- Use Beads only to reconstruct missing context, then write it back to tasks.yaml.
