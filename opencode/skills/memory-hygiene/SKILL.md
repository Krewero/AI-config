---
name: memory-hygiene
description: Rules for safe, consistent, and non-sensitive persistent memory (MCP memory server) across agents and projects.
compatibility: opencode
metadata:
  scope: global
  audience: all-agents
  priority: foundational
---

## What I do
- Keep persistent memory useful (high signal) and safe (no secrets, no client-sensitive data).
- Standardize how we store facts in the Knowledge Graph Memory server: entities, relations, observations.
- Reduce ambiguity about when to write memory vs. when to keep info in the current chat or in project files. 

## When to use me
Use this skill whenever you consider reading or writing to the MCP `memory` server, especially:
- Onboarding a new project/repo and recording stable context.
- Capturing long-lived user preferences (language, tooling, workflow).
- Capturing stable repo conventions (branch naming, release style), as non-sensitive facts.
- Deciding whether something should be persisted or kept ephemeral.

## Core concepts (Knowledge Graph)
- **Entities**: nodes with `name`, `entityType`, and `observations`.
- **Relations**: directed links between entities in active voice (e.g., `uses`, `works_at`).
- **Observations**: atomic strings (one fact per observation) attached to entities.

## Rules
### 1) Safety and privacy: what must NEVER be stored
Do **not** store any of the following in persistent memory:
- Credentials of any kind: passwords, API keys, tokens, certificates, private keys.
- Customer/client proprietary information: internal URLs, usernames, IPs, ticket details, configs, incident logs, business data.
- Personal sensitive data (PII beyond what is necessary for the workflow).

If a user provides any sensitive data, keep it out of memory and, if needed, suggest they rotate/revoke it.

### 2) What SHOULD be stored (high-value, stable, non-sensitive)
Store only information that is:
- Stable over time (likely true next week/month).
- Helpful across sessions.
- Not security-sensitive.

Good examples:
- Preferred language and response format (e.g., “User prefers Italian; concise answers”).  
- Tooling preferences (PowerShell on Windows; uses OpenCode; uses GitHub MCP).  
- Non-sensitive repo conventions (e.g., “Uses Conventional Commits”; “default branch is main”).

### 3) Write memory only when it will be reused
Before writing, ask: “Will this matter outside this chat?”  
If not, keep it in the current conversation context or project docs.

### 4) Naming conventions (to keep the graph clean)
- Entity names must be unique and stable.
- Prefer consistent prefixes for scope, e.g.:
  - `user:<handle>` (user profile)
  - `repo:<owner>/<name>` (repositories)
  - `project:<name>` (project-level)
- Avoid duplicates: search first, then create.

### 5) Observations must be atomic and factual
- One fact per observation string.
- Avoid mixing multiple facts with “and”.
- Avoid speculation; if uncertain, do not store or store as explicitly uncertain (but prefer not storing).

### 6) Relations: active voice and consistent verbs
- Use active voice relation types (e.g., `uses`, `maintains`, `prefers`, `depends_on`).
- Keep relationType consistent across the graph to avoid synonyms (`uses` vs `utilizes`).

### 7) Read before write
- Prefer `search_nodes` for quick lookup and `open_nodes` to retrieve specific entities.
- Use `read_graph` sparingly (it returns the entire graph).

### 8) Minimal set of memory tools (recommended workflow)
When persisting something:
1) `search_nodes` to see if it already exists.
2) `create_entities` only if missing.
3) `add_observations` to append facts.
4) `create_relations` to link entities.

### 9) Correction and cleanup
- If a stored fact becomes outdated or wrong, remove it via `delete_observations` (preferred) or adjust the entity carefully.
- If an entity is clearly erroneous/no longer needed, `delete_entities` is allowed but use cautiously because it deletes relations too.

## Output format (when you write memory)
When you decide to write memory, briefly state:
- What will be stored (1 sentence).
- Why it is useful (1 sentence).
Then perform the memory tool calls.

## Examples
### Example A — user preference
Store: “User prefers Italian, concise answers.”

### Example B — repo convention
Store: “repo:acme/foo uses Conventional Commits; default branch is main.”
