# tasks.yaml Schema (Global)

This file defines the **logical schema** of the `tasks.yaml` file used to delegate work to the AI.

## Overall structure

The following YAML block is an **example schema**, not the actual tasks file.  
Do **not** treat this block as live data; use it only as a template for understanding the format.

```yaml
meta:
  project: "project-name"
  default_branch: "develop"
  owner: "your-name"
  created_at: "YYYY-MM-DD"

workflow:
  mode: "plan-dev-review-organizer"

tasks:
  - id: "T-001"
    title: "Short description of the task"
    description: "the real description"
    status: "TODO"        # TODO | DOING | DONE | BLOCKED | SKIP
    delegable: true       # true if this task can be delegated to the AI
    priority: 1           # optional integer (1 = highest priority)
    paths_hint:
      - "src/main/java/..."
      - "src/test/java/..."
    dont_touch_paths:
      - "db/migrations/"
    conditions:
      - "Specific condition that must be true for this task to be considered done"
      - "Example: no NPE when amount is null"
    rules:
      - "Optional rule specific to this task"
      - "Example: do not change the public signature of PaymentService"
    notes: |
      Free-form text for additional context
      (bug reproduction steps, business notes, links, extra constraints)
    beads: {}  # reserved for Beads metadata (managed by the model)

## Task lifecycle and status updates

A task is considered **completed and accepted** when:

- the implementation is done,
- the code has been reviewed and marked as “OK for commit”,
- I have accepted the changes and (if needed) applied the commit.

In that case, the task’s `status` in `tasks.yaml` should normally be updated to:

- `DONE`: work completed and accepted.

Other possible transitions:

- `TODO` → `BLOCKED`: the task cannot be completed due to external dependencies or constraints.
- `TODO` → `SKIP`: we intentionally decide not to work on this task.

The update of `status` in `tasks.yaml` is done in the **final phase** of the workflow:

- The Repo Organizer proposes how to change the status.
- I explicitly confirm whether the update should be applied.
- Only then is `tasks.yaml` edited to reflect the new status.

## Integration with Beads (optional)

In projects where Beads (`bd`) is used:

- Each task in `tasks.yaml` may have one or more related Beads issues.
- You may reference Beads issue IDs in the `notes` field if helpful, for example:
  - `notes: "Linked Beads issue: bd-a1b2 (epic for this task)"`
- Beads manages long-term, dependency-aware tracking of work,
  while `tasks.yaml` remains the explicit, human-controlled list of tasks delegated to the AI.

In projects where Beads (`bd`) is used, each task may optionally include a `beads` field reserved for Beads-related metadata.

Example:

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
    conditions:
      - "Specific condition that must be true for this task to be considered done"
      - "Example: no NPE when amount is null"
    rules:
      - "Optional rule specific to this task"
      - "Example: do not change the public signature of PaymentService"
    notes: |
      Free-form text for additional context
      (bug reproduction steps, business notes, links, extra constraints)
    beads: {}  # or omitted if not used yet
Rules:

beads is created and updated by the AI/model to store Beads-specific information (for example Beads issue ID, status, sync timestamps, etc.).

The internal structure of beads is free-form and may evolve over time based on how Beads is used.

If beads is absent or empty, the task is simply not linked to Beads.