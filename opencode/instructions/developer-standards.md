# Developer Standards (Global)

These rules apply to any agent that modifies code files (for example, the Developer agent) in any project.

## Scope of changes

- Modify only what is necessary to satisfy the current task.
- Do not:
  - change public APIs without an explicit instruction in the task or prompt,
  - introduce large, unrelated refactors,
  - introduce side effects that are not clearly required by the task.

## Diff and granularity

- Changes must be structured into clear logical blocks.
- The diff should be:
  - consistent with `git diff`,
  - readable,
  - understandable even without opening the files.
- Each logical block of changes should have a short explanation (“rationale”).

## Error handling and logging

- Handle edge cases (nulls, empty collections, external errors) when they are relevant to the task.
- Do not silently swallow significant exceptions.
- Use logging consistent with the project’s conventions (levels, formatting).

## Performance and scalability

- Avoid N+1 queries and unnecessary loops when touching code that interacts with databases or remote services.
- Avoid unnecessary allocations in hot paths (for example, loops over large collections).
- Do not introduce complex new data structures unless strictly necessary.

## Style and consistency

- Adapt to the existing style of the file/project (naming, formatting, patterns).
- Do not impose new global conventions out of nowhere.
- When proposing a structural improvement, clearly explain the benefit (readability, performance, maintainability).
