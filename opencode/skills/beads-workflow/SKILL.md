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
- Maintain a standard `beads: {}` section in `tasks.yaml` so mapping between tasks and Beads issues is unambiguous.

## When to use me
- The `opencode-beads` plugin is enabled.
- A task moves to `DOING` or `BLOCKED` and should survive crashes / context loss.
- Recovering execution context for DOING/BLOCKED tasks after an incident.

## Task identity and mapping
- Every task must have a stable `id` field stored in `tasks.yaml` (e.g. `id: T-0001`).
- The `id` must not change over the lifetime of the task.
- Beads issues are linked to tasks via this `id`.

## Standard beads block in tasks.yaml
`tasks.yaml` (at repo root) MUST contain a top-level `beads:` section.

Recommended minimal structure:

```yaml
beads:
  enabled: true
  mode: "ai-only"
  mapping:
    by_task_id: {} # task_id -> bead_id (string)
  sync:
    last_sync_at: null # ISO timestamp
    last_error: null
