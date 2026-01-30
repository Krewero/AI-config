---
name: beads-workflow
description: AI-only Beads workflow. tasks.yaml in repo root is the source of truth; Beads mirrors DOING/BLOCKED tasks to provide persistent, recoverable execution context.
compatibility: opencode
metadata:
  scope: global
  audience: ai-agents
---

## What I do
- Use Beads only as an AI-managed persistence layer for in-flight tasks.
- Keep `./tasks.yaml` as the single source of truth for task state and planning.
- Maintain beads metadata inside each task object to map tasks to Beads issues unambiguously.

## When to use me
- The `opencode-beads` plugin is enabled.
- A task moves to `DOING` or `BLOCKED` and should survive crashes / context loss.
- Recovering execution context for DOING/BLOCKED tasks after an incident.

## Task identity and mapping
- Every task must have a stable `id` field stored in `tasks.yaml` (e.g. `id: T-0001`).
- The `id` must not change over the lifetime of the task.
- Beads issues are linked to tasks via the `beads` metadata block inside each task.

## Standard beads block in tasks.yaml (inside each task)
Each task in `tasks.yaml` MUST contain a `beads:` section with this structure:

```yaml
tasks:
  - id: T-0001
    title: "Task title"
    ...
    beads:                # AI-managed metadata for task persistence
      bead_id: null       # Beads issue ID linked to this task (string when mirrored, else null)
      mirrored: false     # true when task is mirrored in Beads (DOING/BLOCKED status)
      last_sync_at: null  # ISO timestamp of last beads sync
      last_error: null    # Last error encountered during beads sync (if any)
      session_id: null    # Session ID currently working on this task
      agent: null         # Agent name that last worked on this task
```

All beads fields default to `null` (or `false` for `mirrored`) when task is created.

## Mirroring policy
- Mirror ONLY tasks with status: `DOING` or `BLOCKED`.
- Do NOT mirror tasks with status: `TODO`, `DONE`, or `SKIP`.
- Update `task.beads.mirrored` accordingly.

## What can be stored in external Beads issues
**Allowed** (recovery-oriented only):
- Immediate next action
- Copy-paste commands to run (do not execute)
- Links: files, commits, PRs, issues
- Blockers and required inputs

**Forbidden**:
- Secrets/tokens/credentials
- Client proprietary or sensitive operational data

## Task_Manager responsibilities
**Task_Manager MUST:**
- Enforce schema and keep tasks.yaml consistent.
- When a task becomes `DOING` or `BLOCKED`:
  - Create or update the external Beads issue (AI-only).
  - Update `task.beads` fields: `bead_id`, `mirrored` (true), `last_sync_at`, `session_id`, `agent`.
- When a task becomes `DONE` or `SKIP`:
  - Update status in tasks.yaml.
  - Leave `task.beads` fields as-is for historical tracking.
  - Optionally leave the external Beads issue as historical context.

**Task_Manager MUST NOT:**
- Change task status based on Beads alone.
- Store sensitive info in Beads or in tasks.yaml.

## Incident recovery procedure
When recovering from an incident:
1. Read `./tasks.yaml` first (authoritative).
2. Identify tasks in `DOING` or `BLOCKED` status.
3. For each in-flight task:
   - Check `task.beads.bead_id` to find linked Beads issue.
   - Recover missing execution context from external Beads issue.
   - Write recovered context into `task.notes` or other task fields.
4. Continue work with tasks.yaml as authoritative.

## Conflict resolution
If external Beads and tasks.yaml disagree:
- **tasks.yaml wins** for status and task definition.
- Use Beads only to reconstruct missing context, then write it back to tasks.yaml.