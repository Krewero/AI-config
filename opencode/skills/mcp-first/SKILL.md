---
name: mcp-first
description: Prefer MCP tools (git*/github*) for repo and GitHub actions; use bash only as an explicit fallback when MCP is unavailable.
compatibility: opencode
metadata:
  scope: global
  audience: all-agents
  priority: foundational
---

## What I do
- Make tool selection deterministic: prefer `git*` and `github*` MCP tools over `bash` for Git/GitHub tasks.
- Define a clear fallback path when MCP is unavailable or failing.
- Reduce ambiguity between "inspect" vs "change" actions so agents don't accidentally execute shell commands.

## When to use me
Use this skill whenever the user asks for anything related to:
- GitHub: repositories, issues, pull requests, users, branches, releases.
- Git: status, diff, log, branch, tags, commits, file history.

## Rules
### 1) Default to MCP for Git/GitHub
- For Git operations, use `git*` MCP tools first.
- For GitHub operations, use `github*` MCP tools first.

### 2) Use bash only as fallback (and be explicit)
- Use `bash` only if MCP is unavailable, failing, or missing a capability required for the request.
- If falling back to bash, explain which MCP capability was missing (one short sentence).
- Prefer read-only shell commands unless the user explicitly asked for changes.

### 3) Never execute "write" Git operations implicitly
- Do not run destructive or state-changing Git commands via `bash` unless explicitly requested.
- If the user asks to "push", "merge", "rebase", "tag", or "release", propose copy-paste commands and ask for confirmation before any execution.

### 4) Prefer inspection before action
Before proposing any change commands, gather context:
- repo status, current branch, uncommitted changes, pending commits, remote name.  
If MCP is available, gather this context via MCP tools; otherwise via safe bash commands.

### 5) Output format for shell fallback
When you must propose bash commands:
- Provide a short "Commands" code block.
- Keep commands copy-pasteable and ordered.
- Do not add commentary inside the code block.

## Examples
### Example A — list repositories (GitHub)
Goal: "List my repositories"
- Use `github*` MCP tools.
- Do not use `gh` CLI or `bash` unless MCP is down.

### Example B — propose a safe push workflow (Git)
Goal: "Push my changes"
- Prefer MCP for inspection (status/diff/log) first.
- Then propose copy-paste commands for add/commit/push.
- Do not execute them unless explicitly asked.
