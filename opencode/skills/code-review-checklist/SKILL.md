---
name: code-review-checklist
description: Standard code review checklist for correctness, maintainability, performance, and operability. Provide APPROVE/REJECT decision with actionable feedback.
compatibility: opencode
metadata:
  scope: global
  audience: all-agents
  workflow: review
---

## What I do
- Provide a consistent, low-ambiguity code review checklist.
- Focus on correctness, maintainability, performance, and operability.
- Issue a clear **APPROVE** or **REJECT** decision based on checklist results.
- Produce actionable feedback (what/why/how) without making code changes automatically.

## When to use me
Use this skill when:
- Reviewing a diff, PR, or patch.
- Assessing code quality before merge.
- The user asks: "review this", "is this safe?", "any issues?", "what would you improve?"

## Review output format (required)
When reviewing, structure feedback as:

1. **Decision:** `APPROVE` or `REJECT`
2. **Checklist Results:** Mark each section ✅ (pass) or ❌ (fail)
3. **Strengths:** 2-3 positive highlights (what was done well)
4. **Weaknesses:** Specific issues with file/line references (only if failed)
5. **Actionable Feedback:** What to fix if rejected (blocker items first)

### Decision Criteria
- **APPROVE:** All critical checklist items pass; minor suggestions are acceptable.
- **REJECT:** One or more critical issues (blockers) found in checklist.

### Feedback Categories
For each issue, include:
- **Blockers** (must fix) - Critical issues preventing approval
- **Warnings** (should fix) - Important issues, not blocking but risky
- **Suggestions** (nice to have) - Improvements for code quality
- **Questions** (clarifications needed) - Unclear intent or behavior

For each item, provide:
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

### H) Config & deployment readiness
- Config changes are documented and have safe defaults.
- Feature flags are used for risky changes where appropriate.
- Migration steps are clearly stated (DB migrations, env vars, runtime requirements).

## Rules (non-negotiable)
- Do not execute commands or modify files as part of code review unless explicitly requested.
- If you cannot verify something from the diff/context, ask a question instead of guessing.
- Prefer fewer, high-signal comments over many minor nits.
- Always issue a clear APPROVE or REJECT decision.

## Example Output

### Example 1: APPROVE

```
Decision: APPROVE ✅

Checklist Results:
✅ Correctness
✅ Reliability & error handling
✅ Maintainability
✅ Performance
✅ Observability
✅ Config & deployment readiness

Strengths:
- Clean separation of concerns in the service layer
- Comprehensive error handling with appropriate status codes
- Good test coverage for edge cases

Suggestions:
- Consider extracting the validation logic into a separate validator class for reusability
```

### Example 2: REJECT

```
Decision: REJECT ❌

Checklist Results:
✅ Correctness
❌ Reliability & error handling
✅ Maintainability
✅ Performance
✅ Observability
✅ Config & deployment readiness

Weaknesses (Blockers):
- [src/api/UserController.java:45] Raw exception messages exposed to API clients
  Impact: Sensitive internal information (stack traces, DB paths) leaked
  Fix: Map exceptions to sanitized error responses; log full details server-side only

- [src/service/PaymentService.java:89] Missing input validation on amount field
  Impact: Negative amounts could bypass business logic
  Fix: Add validation: amount > 0 and within acceptable range

Actionable Feedback:
- Sanitize all exception messages in API responses (blocker)
- Add amount validation in PaymentService (blocker)
- After fixes, re-run review
```

## Example wording (recommended)
- **Blocker:** "This endpoint returns raw exception messages; this can leak sensitive info. Please map exceptions to sanitized error responses and log details server-side."
- **Warning:** "This query could cause N+1 problem with large datasets. Consider using a JOIN or batch loading."
- **Suggestion:** "Consider extracting this validation into a helper to reduce duplication and simplify testing."