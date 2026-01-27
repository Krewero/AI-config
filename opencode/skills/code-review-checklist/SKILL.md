---
name: code-review-checklist
description: Standard code review checklist for correctness, security, maintainability, performance, and operability. Provide actionable feedback and never apply changes automatically.
compatibility: opencode
metadata:
  scope: global
  audience: all-agents
  workflow: review
---

## What I do
- Provide a consistent, low-ambiguity code review checklist.
- Focus on correctness, security, maintainability, performance, and operability.
- Produce actionable feedback (what/why/how) without making code changes automatically.

## When to use me
Use this skill when:
- Reviewing a diff, PR, or patch.
- Assessing code quality before merge.
- The user asks: "review this", "is this safe?", "any issues?", "what would you improve?"

## Review output format (required)
When reviewing, structure feedback as:
1) **Blockers** (must fix)
2) **Warnings** (should fix)
3) **Suggestions** (nice to have)
4) **Questions** (clarifications needed)

For each item, include:
- Location (file + function or line range if available)
- Issue (what is wrong)
- Impact (why it matters)
- Fix (how to address it)

## Checklist

### A) Correctness
- Are edge cases handled (null/empty, boundaries, error paths)?
- Are there race conditions or ordering assumptions?
- Are there off-by-one or timezone/locale issues (dates, money, rounding)?
- Is behavior deterministic where it needs to be?

### B) Security (baseline)
- No secrets in code, logs, configs, or error messages.
- Input validation: validate at boundaries (API, DB, queue, file).
- Authorization: verify access control checks are present and correct.
- Data exposure: avoid leaking internal identifiers and stack traces to clients.
- Safe dependencies: new packages are justified and versions are constrained.

### C) Reliability & error handling
- Errors are handled intentionally (retry vs fail-fast).
- Exceptions include enough context for debugging but no sensitive data.
- Timeouts are set for network calls.
- Idempotency is considered for handlers that may be retried.

### D) Maintainability
- Naming is clear and consistent.
- Functions are small and single-purpose.
- Complex logic is explained with minimal comments (why, not what).
- No duplication: shared logic is extracted appropriately.

### E) Performance
- Hot paths avoid unnecessary allocations and repeated work.
- DB access avoids N+1 patterns; queries are bounded.
- Caching is correct (cache key, invalidation, TTL) where used.
- Pagination/limits exist for potentially large results.

### F) Observability (logs/metrics/tracing)
- Logs are meaningful and structured (if the project supports it).
- Log levels are appropriate (debug/info/warn/error).
- Correlation IDs / request IDs are propagated where relevant.
- Metrics/tracing are added for important operations (if standard in repo).

### G) API & contracts
- Public APIs keep backward compatibility (or there is a migration plan).
- Contracts are documented (request/response, schemas, validation rules).
- Versioning is handled when changes are breaking.

### H) Config & deployment readiness
- Config changes are documented and have safe defaults.
- Feature flags are used for risky changes where appropriate.
- Migration steps are clearly stated (DB migrations, env vars, runtime requirements).

## Rules (non-negotiable)
- Do not execute commands or modify files as part of code review unless explicitly requested.
- If you cannot verify something from the diff/context, ask a question instead of guessing.
- Prefer fewer, high-signal comments over many minor nits.

## Example wording (recommended)
- Blocker: “This endpoint returns raw exception messages; this can leak sensitive info. Please map exceptions to sanitized error responses and log details server-side.”
- Suggestion: “Consider extracting this validation into a helper to reduce duplication and simplify testing.”
