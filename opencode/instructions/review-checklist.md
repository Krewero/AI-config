# Code Review Checklist (Global)

This checklist guides the Reviewer agent when analyzing code changes.

## Severity levels

Use these categories:

- `CRITICAL`: a bug or security issue that is likely or certain.
- `MAJOR`: a serious issue in correctness, performance or design that must be fixed before commit.
- `MINOR`: readability, style or small optimization improvements.
- `INFO`: non-binding observations or suggestions.

## Base checklist

For every change, check:

1. **Correctness**
   - Does the logic actually do what the task requires?
   - Are obvious edge cases (nulls, empty collections, external errors) handled when relevant?

2. **Security**
   - Are invalid or malicious inputs handled appropriately?
   - Are secrets or sensitive data avoided in logs?
   - Are unnecessary internal details avoided in external outputs?

3. **Performance**
   - Are there new loops or iterations that are unnecessary?
   - Has the number of database or remote service calls increased?
   - Is the overall complexity acceptable for the context?

4. **Structure and maintainability**
   - Is the code readable and consistent with the project’s style?
   - Has the method/class become too complex?
   - Are there duplications that could have been avoided?

5. **Scope**
   - Are the changes limited to the current task?
   - Are there unrelated changes (unnecessary refactors, renames, etc.)?

## Expected Reviewer output

- A numbered list of findings, each with:
  - a severity (`CRITICAL` / `MAJOR` / `MINOR` / `INFO`),
  - a clear description,
  - an optional concrete suggestion.
- If there are `CRITICAL` or `MAJOR` issues, the conclusion must be “Not OK for commit”.
- If there are only `MINOR` or `INFO` issues, the conclusion can be “OK for commit” with recommendations.
