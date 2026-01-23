# Workflow: Plan → Developer → Reviewer → Repo Organizer

This file describes the **global workflow** that agents should follow when working with `tasks.yaml`.

## Key concepts

- Always work on **one task at a time**.
- The primary source of tasks is `tasks.yaml` in the project root.
- Valid task statuses are: `TODO`, `DOING`, `DONE`, `BLOCKED`, `SKIP`.
- The AI does **not** run CLI commands: it only proposes commands and procedures.
- When Beads (`bd`) is available in the project, you may use it as **long-term memory**:
  - use Beads issues to track detailed work, discovered tasks and dependencies,
  - but always treat `tasks.yaml` as the source of truth for which task is actively delegated in this session.


## Task selection

When `tasks.yaml` exists:

1. Read `tasks.yaml`.
2. Select the **first task** that has:
   - `status: TODO`
   - `delegable: true`
3. If there are no delegable tasks, ask me what to do next.

## Plan phase (requirements and design)

**Responsible agent:** the **Plan** agent.

Role: analyze the selected task and propose a technical plan.

The Plan agent may automatically call the **Developer** subagent to perform the implementation phase for the selected task in the same conversation, after finishing the analysis.

Input:
- The selected task from `tasks.yaml`.
- Any `conditions` present in the task.
- Global rules in `AGENTS.md` and in other instruction files.

Output (text only, no file changes, no commands):

- A summary of the task (2–5 sentences).
- A list of technical micro‑tasks (1..N) to implement it.
- Candidate files to read/modify (based on `paths_hint` and the project).
- Main risks or edge cases to consider.
- Tests that should be written or updated (as suggestions only).

The Plan phase **must not**:
- modify files,
- execute commands,
- access unnecessary external resources.

### Beads integration (optional)

If Beads (`bd`) is available in this project:

- Check whether there is already a Beads issue linked to this task (for example by title or by explicit reference in `tasks.yaml` or `notes`).
- If there is no matching issue and this task is large or non-trivial, you may propose creating a new Beads issue for it (epic or task).
- When you create or reference a Beads issue, clearly mention the relationship between:
  - the `tasks.yaml` task (its `id`) and
  - the Beads issue ID (for example `bd-a1b2`).

Do not rely on Beads alone to decide what to work on: always start from `tasks.yaml`.

## Developer phase (implementation)

**Responsible agent:** the **Developer** agent.

Role: implement the selected task by changing files and producing diffs.

Input:
- The selected task from `tasks.yaml`.
- The plan produced by the Plan phase.
- Rules in `developer-standards.md` and `tasks-schema.md`.

Behavior:

- Work on a single task at a time.
- Respect any:
  - `paths_hint`
  - `dont_touch_paths`
- Make changes directly to the files (when allowed by the agent’s permissions).
- Always produce a **human‑readable diff** (similar to `git diff`) in the response.

Required output:

1. A list of modified files.
2. A short explanation of what changed and why.
3. A textual diff of the changes.

The Developer phase **must not** run CLI commands and **must not** perform commits.

### Beads integration (optional)

If Beads (`bd`) is available in this project:

- While implementing a task from `tasks.yaml`, you may:
  - update the status of the corresponding Beads issue (for example `in_progress`, `done`, `blocked`),
  - create new Beads issues for bugs, follow-up work or refactors discovered during implementation,
  - link new issues to the current Beads issue using dependencies (for example `discovered-from` or `parent-child`).
- When you pause or stop work (for example because I have to shut down the machine), make sure the Beads issue(s) reflect the current state of the work so that you can resume later without me re-explaining everything.

Even when Beads is used, you must still respect the current task’s `status` in `tasks.yaml` and the rules in this workflow.

## Reviewer phase (code review)

**Responsible agent:** the **Reviewer** agent.

Role: verify the quality of the changes (both AI‑generated and human‑written).

Input:
- The modified code (files and/or diff).
- Rules in `review-checklist.md`.

Required output:

- A bullet list of findings, each with a **severity** (such as `CRITICAL`, `MAJOR`, `MINOR`, `INFO`).
- Specific comments that refer to files/lines or diff sections.
- Concrete improvement suggestions.
- A final judgment:
  - “OK for commit” or
  - “Not OK for commit” with a clear reason.

The Reviewer is **read‑only**: it does not modify files and does not execute commands.

## Repo Organizer phase (commit procedure)

**Responsible agent:** the **Repo Organizer** agent.

Role: provide a clear **CLI procedure** to commit the changes in:

- my personal Git repository,
- the company SVN repository,
- and help keep `tasks.yaml` in sync with the actual state of the work.

Input:
- The list of modified files.
- A short description of the task and changes.
- The current entry for this task in `tasks.yaml`.

Required output:

1. **Git commands** (in order), for example:
   - `git status`
   - `git diff` (optionally with paths)
   - `git add ...`
   - `git commit -m "<message>"`
   - `git push` (when appropriate)
2. **SVN commands** (in order), for example:
   - `svn status`
   - `svn diff`
   - `svn add ...` (if there are new files)
   - `svn commit -m "<message>"`
3. Rules for commit messages:
   - Short and descriptive.
   - Prefer a conventional style when possible, for example:
     - `fix: handle null amount in PaymentService`
     - `refactor: simplify validate() logic in XService`
     - `chore: update configuration for payment retries`
4. A pre‑commit checklist:
   - Confirm the diff contains only what the task requires.
   - Confirm the relevant tests (if any) have been executed and are passing.
   - Confirm no temporary or build files are accidentally tracked.
5. **tasks.yaml update proposal**:
   - Propose how to update the current task in `tasks.yaml` **after I confirm that the work is accepted**.
   - Typical transitions:
     - `status: TODO` → `status: DONE` when the task is fully completed and reviewed.
     - `status: TODO` → `status: BLOCKED` if the task cannot proceed due to external blockers.
     - `status: TODO` → `status: SKIP` if we decide to skip the task.
   - Do not actually edit `tasks.yaml` unless I explicitly confirm that you can update it.

The Repo Organizer **must not** execute commands: it only produces text and procedures.  
It may be allowed to edit `tasks.yaml` in a very controlled way, but only after my explicit approval.