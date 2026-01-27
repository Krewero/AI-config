---
name: task-workflow
description: Standard workflow for managing tasks.yaml (create/update/status), keeping tasks atomic, actionable, and traceable across sessions.
compatibility: opencode
metadata:
  scope: global
  audience: all-agents
  priority: foundational
---

## What I do
- Provide a consistent, low-ambiguity process to create, update, and close tasks in `tasks.yaml`.
- Keep tasks atomic, actionable, and easy to track (clear status, clear next step).
- Define a shared meaning for statuses and a consistent “Definition of Done”.

## When to use me
Use this skill whenever:
- The user asks to create or update tasks.
- The agent needs to transform a goal into actionable work items.
- The work is multi-step and should be tracked across sessions.

## Task statuses
Use only these statuses (unless the repo already defines others):
- `TODO`: Not started, no active work.
- `DOING`: Actively being worked on (there should be a clear next action).
- `BLOCKED`: Cannot proceed due to a dependency or missing input.
- `DONE`: Completed and verified against acceptance criteria.
- `SKIP`: Intentionally not doing it (with a reason).

## Rules
### 1) Tasks must be atomic and testable
- One task = one deliverable.
- A task must be verifiable (“I can tell if it’s done”).
- If a task includes multiple deliverables, split it.

### 2) Always capture acceptance criteria (DoD)
Each task must include:
- **Outcome**: what changes in the system.
- **Validation**: how to verify it (tests, command, manual check).

If validation is unclear, ask one clarifying question before writing tasks.

### 3) Use context links for traceability
Whenever possible, attach references:
- File paths (e.g., `src/...`)
- PR / Issue links (GitHub)
- Commands to reproduce (copy-paste, not executed)

### 4) Status transitions must be explicit
Allowed transitions:
- `TODO` → `DOING`
- `DOING` → `DONE`
- `DOING` → `BLOCKED`
- `BLOCKED` → `DOING`
- Any → `SKIP` (only with a reason)

Do not move multiple tasks to `DOING` at the same time unless the user asked for parallel work.

### 5) Notes must capture decisions, not chatter
Good notes:
- Constraints discovered, decisions made, trade-offs.
Bad notes:
- Long conversational text, irrelevant background.

### 6) Prefer updating tasks over creating duplicates
Before creating new tasks:
- Search existing tasks for similar items.
- If it’s the same work, update the existing one (add details, change status, add notes).

### 7) Keep task titles short and imperative
Good: “Add GitHub MCP auth header support”  
Bad: “We should probably look into making GitHub MCP work somehow”

### 8) If blocked, record what is needed
When setting `BLOCKED`, add:
- What is blocking.
- What input is needed (and from whom).
- The next action once unblocked.

## Recommended output format (in chat)
When proposing task updates:
1) A short list of tasks to create/update (titles + intended status).
2) Ask up to 1–2 clarifying questions if required.
3) Then update `tasks.yaml` (if you have write/edit permissions) or provide a patch.

## Example
Goal: “Enable persistent memory MCP for the project”
- Task 1 (TODO): “Add memory MCP server config to opencode.jsonc”; Validation: `opencode mcp list` shows `memory connected`.
- Task 2 (TODO): “Add memory hygiene rules”; Validation: skill `memory-hygiene` exists and is loadable.

## Beads integration (incident-only persistence)
- `tasks.yaml` is the single source of truth for task status, notes, and acceptance criteria.
- If the `opencode-beads` plugin is enabled, Beads is used only as an emergency persistence/continuity layer (e.g., incident recovery, context loss), not as the day-to-day task manager.
- Do not create a parallel workflow where the same task is actively managed in both places.
- If Beads is used during recovery, treat it as a reference to reconstruct `tasks.yaml`, then make `tasks.yaml` authoritative again.
- When there is a conflict between Beads and `tasks.yaml`, assume `tasks.yaml` is correct and update any recovered notes accordingly.