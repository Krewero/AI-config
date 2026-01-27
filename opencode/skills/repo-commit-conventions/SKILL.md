---
name: repo-commit-conventions
description: Standard commit workflow and message conventions (copy-paste commands). Prefer MCP for inspection; never push/merge automatically.
compatibility: opencode
metadata:
  scope: global
  audience: all-agents
  workflow: git
---

## What I do
- Make Git commit workflows consistent and low-ambiguity across projects.
- Prefer MCP (`git*` / `github*`) for repository inspection when available.
- Provide copy-pasteable command sequences; do not execute destructive commands implicitly.

## When to use me
Use this skill when the user asks for:
- Commit message suggestions.
- A safe sequence of Git commands (status/diff/add/commit/push).
- PR-related conventions (branch naming, linking PRs/issues in commits).
- “Prepare my changes for review” or “What should I commit?”

## Core rules
### 1) Never run a push/merge/rebase/tag automatically
- Always propose commands; do not execute them unless explicitly requested.
- If execution is requested, ask for confirmation before running any state-changing commands. [page:1]

### 2) Inspect before committing
Before proposing any commit, gather:
- Current branch name.
- Working tree status (staged/unstaged).
- Diff summary (what changed).
- Recent commits (to avoid duplicates).

Prefer MCP tools for these reads; fall back to safe bash commands only if needed.

### 3) Conventional Commits format
Use Conventional Commits unless the repo explicitly uses something else:

`<type>(<scope>): <subject>`

Rules:
- `type` must be one of: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `build`, `ci`, `perf`, `style`, `revert`.
- `scope` is optional; use it when it reduces ambiguity (module/service name).
- `subject` is imperative, present tense, no trailing period, <= 72 chars.

### 4) Commit granularity
- Prefer multiple small commits over one large mixed commit.
- Do not mix refactors and behavior changes in the same commit.
- Avoid committing unrelated formatting changes together with logic changes.

### 5) Linking work items (when available)
If there is an issue/PR/task reference:
- Add it in the commit body (preferred) rather than the subject.
- If the repository enforces a specific pattern, follow that.

### 6) Branch naming (default)
If branch naming is not specified by the repo:
- `feat/<short-slug>`
- `fix/<short-slug>`
- `chore/<short-slug>`

Keep slugs lowercase, hyphen-separated.

