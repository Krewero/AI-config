# OpenCode Agent System

## Purpose
This file defines the foundational principles and organizational structure for the OpenCode agent ecosystem.
It serves as the entry point for understanding how agents operate, coordinate, and maintain system integrity.

---

## Core philosophy

### 1) Task-driven development
All work is organized around tasks stored in a canonical task file at the repository root.
This task file is the single source of truth for:
- What work exists
- Current status of each work item
- Acceptance criteria and constraints
- Safety boundaries

### 2) Agent specialization
Agents are specialized by role:
- Some agents plan and analyze without making changes.
- Some agents create and maintain task definitions.
- Some agents implement code changes.
- Some agents review quality and correctness.
- Some agents prepare versioning commands without executing them.
- Some agents execute explicit user commands.

Each agent operates within its designated scope and respects boundaries defined by configuration.

### 3) Copy-paste first for destructive operations
Agents propose commands for version control, deployment, and other potentially destructive operations.
Users execute these commands manually unless explicitly granting execution permission.

### 4) Persistent context (AI-managed)
Task state persistence is managed by specialized tooling to enable recovery from crashes, session loss, or context compaction.
This persistence is AI-only; humans interact with the canonical task file, not with persistence layers.

### 5) MCP-first tooling
When interacting with external systems (version control, issue tracking, knowledge graphs), agents prefer dedicated tool protocols over shell commands.
Shell commands are used as fallback only when dedicated tools are unavailable.

---

## Workflow structure

### Initialization
Projects are initialized with foundational files and structures before task-based work begins.

### Task lifecycle
1. Tasks are created from user input and stored in the canonical task file.
2. Tasks move through defined states (not started, in progress, blocked, completed, skipped).
3. Agents specialized for planning analyze tasks and create implementation strategies.
4. Agents specialized for implementation execute changes following those strategies.
5. Agents specialized for review validate changes against acceptance criteria and best practices.
6. Task state is updated to reflect completion or required corrections.
7. Versioning commands are generated for review and manual execution.

### Background synchronization
Throughout the workflow, task state and metadata are kept synchronized with persistence layers to ensure recovery capability.

---

## Safety and boundaries

### Path safety
Tasks may define paths that must not be modified.
Agents must respect these boundaries and stop if changes are required in restricted areas.

### Delegation control
Tasks may be marked as non-delegable to the AI.
For non-delegable tasks, agents may only propose solutions, ask clarifying questions, or document what a human should do.

### Sensitive data
Agents must never persist:
- Credentials, tokens, API keys, certificates, or passwords
- Client-proprietary or commercially sensitive information
- Personal identifying information beyond what is necessary for workflow context

If sensitive data is encountered, it must be kept out of all persistence layers and the user should be advised to rotate/revoke it.

---

## Agent coordination

### Default behavior
Work begins with planning and analysis.
Changes are not made until deliberate handoff to specialized implementation agents.

### Explicit handoff
Agents hand off work to specialized peers when their role boundary is reached:
- Planning agents hand off to implementation agents when plans are ready.
- Implementation agents hand off to review agents when changes are complete.
- Review agents hand off back to planning if corrections are needed.
- Version control preparation agents hand off to execution agents (or users) for final command execution.

### State synchronization
Task management agents maintain the canonical task file and synchronize with persistence layers.
These agents operate in the background during workflow execution to keep state consistent.

---

## Configuration and customization

### Global vs. project-local
- Global configuration defines baseline behavior, tool preferences, and agent capabilities.
- Project-local configuration may override globals for project-specific requirements (technology stack, conventions, constraints).

### Instruction hierarchy
Detailed operational knowledge is organized into focused instruction documents:
- Technology stack and framework conventions
- Complete workflow phase descriptions
- Task system structure and persistence rules

These documents are loaded via configuration and are not referenced directly in this file to avoid coupling and maintenance burden.

---

## Technology assumptions (overridable per project)

### Backend
Java-based frameworks with dependency management via Maven.
Data access via ORM and query mapping libraries.

### Frontend
Modern component-based JavaScript frameworks.

### Databases
Relational databases (enterprise and open-source options).

### Local development
Containerized services for isolation and reproducibility.

### CI/CD
Pipeline-based continuous integration.
Container orchestration for cloud deployment.

### Version control
Git for personal projects, centralized VCS for company projects.

Projects may override these assumptions via project-local configuration.

---

## Summary

This agent system is designed for:
- **Clarity**: Single source of truth for task state; clear agent roles.
- **Safety**: Boundaries on file modification, delegation control, no secret persistence.
- **Resilience**: AI-managed persistence for crash recovery without human intervention.
- **Flexibility**: Global defaults with project-local overrides.
- **Auditability**: Task history and state transitions are explicitly tracked.

Agents should operate within these principles and defer to specialized instruction documents for detailed operational rules.
