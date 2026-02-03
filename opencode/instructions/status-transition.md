# Status Transitions (Task Workflow)

## Purpose
This document defines the valid status transitions for tasks in `tasks.yaml` and specifies who is responsible for each transition.

## Valid Task Statuses
- **TODO**: Task is defined but not started
- **DOING**: Task is actively being worked on
- **BLOCKED**: Task cannot proceed (dependency, missing input, blocker)
- **DONE**: Task is completed and validated
- **SKIP**: Task is intentionally not being implemented (with reason)

---

## Status Transition Rules

### TODO → DOING
**When**: Planning starts for a task
**Who**: Task_Manager
**Trigger**: OCA delegates `/plan` command to Task_Manager
**Actions**:
  1. Task_Manager validates task (exists, TODO, delegable)
  2. Updates status: TODO → DOING
  3. Mirrors to Beads (creates Beads issue)
  4. Updates beads metadata (bead_id, mirrored=true, timestamps)
  5. OCA then delegates to plan agent for plan creation

**Example**:
```
User: /plan T-0042
→ OCA validates → Task_Manager updates T-0042: TODO → DOING
→ OCA delegates to plan → plan creates ./plans/plan-T-0042.md
```

---

### DOING → DONE
**When**: Task implementation is complete and approved
**Who**: Task_Manager
**Trigger**: Code review approves OR user marks complete
**Actions**:
  1. Task_Manager updates status: DOING → DONE
  2. Closes Beads issue (delegates to @beads-task-agent)
  3. Updates beads metadata (keep for history, update last_sync_at)

**Example**:
```
User: /review → Code_Reviewer: APPROVE
→ User confirms task complete → Task_Manager: T-0042 DOING → DONE
→ Beads issue closed
```

---

### DOING → BLOCKED
**When**: Task cannot proceed due to dependency, blocker, or missing input
**Who**: Task_Manager
**Trigger**: build agent encounters blocker OR user reports blocker
**Actions**:
  1. Task_Manager updates status: DOING → BLOCKED
  2. Appends blocker description to notes
  3. Updates Beads issue with blocker details
  4. Updates beads metadata (last_sync_at)

**Example**:
```
build agent: "Missing dependency: spring-security"
→ Task_Manager: T-0042 DOING → BLOCKED
→ notes += "BLOCKED: Missing spring-security dependency"
→ Beads issue updated with blocker
```

---

### BLOCKED → DOING
**When**: Blocker is resolved, dependency available, input provided
**Who**: Task_Manager
**Trigger**: User resolves blocker and resumes work
**Actions**:
  1. Task_Manager updates status: BLOCKED → DOING
  2. Optionally appends resolution note
  3. Updates Beads issue with resolution
  4. Updates beads metadata (last_sync_at)

**Example**:
```
User: "Added spring-security dependency, unblock T-0042"
→ Task_Manager: T-0042 BLOCKED → DOING
→ notes += "UNBLOCKED: spring-security dependency added"
→ Beads issue updated
```

---

### Any → SKIP
**When**: Task is intentionally not being implemented
**Who**: Task_Manager
**Trigger**: User decides to skip task
**Actions**:
  1. Task_Manager updates status: [any] → SKIP
  2. Adds reason to notes (mandatory)
  3. Closes Beads issue if mirrored
  4. Updates beads metadata (keep for history)

**Example**:
```
User: "Skip T-0042, feature no longer needed"
→ Task_Manager: T-0042 DOING → SKIP
→ notes += "SKIPPED: Feature no longer needed per product decision"
→ Beads issue closed
```

---

## Invalid Transitions

### TODO → DONE (invalid)
**Why**: Tasks must go through DOING to track active work
**Correct flow**: TODO → DOING → DONE

### DONE → any status (invalid)
**Why**: DONE is a final state, cannot reopen
**If task needs rework**: Create new task with reference to completed one

### SKIP → any status (invalid)
**Why**: SKIP is a final state, cannot reopen
**If task is needed later**: Create new task with reference to skipped one

---

## Transition Matrix

| From    | To      | Valid? | Who          | Notes                          |
|---------|---------|--------|--------------|--------------------------------|
| TODO    | DOING   | ✅     | Task_Manager | Planning starts                |
| TODO    | SKIP    | ✅     | Task_Manager | Decided not to implement       |
| DOING   | DONE    | ✅     | Task_Manager | Implementation complete        |
| DOING   | BLOCKED | ✅     | Task_Manager | Blocker encountered            |
| DOING   | SKIP    | ✅     | Task_Manager | Decided to abandon             |
| BLOCKED | DOING   | ✅     | Task_Manager | Blocker resolved               |
| BLOCKED | SKIP    | ✅     | Task_Manager | Decided to abandon             |
| TODO    | DONE    | ❌     | -            | Must go through DOING          |
| DONE    | any     | ❌     | -            | Final state, cannot reopen     |
| SKIP    | any     | ❌     | -            | Final state, cannot reopen     |

---

## Beads Integration

### When to Mirror (Create Beads Issue)
- Task transitions to DOING or BLOCKED
- Task is not yet mirrored (beads.mirrored = false)
- Task_Manager delegates to @beads-task-agent to create issue
- Updates task.beads metadata in tasks.yaml

### When to Update Beads Issue
- Task remains in DOING or BLOCKED but notes/context change
- Task_Manager delegates to @beads-task-agent to update issue
- Updates task.beads.last_sync_at

### When to Close Beads Issue
- Task transitions to DONE or SKIP
- Task_Manager delegates to @beads-task-agent to close issue
- Keeps task.beads metadata as historical record

---

## Agent Responsibilities

### OCA (Orchestrator)
- Validates workflow prerequisites
- Delegates /plan to Task_Manager (for status update) then plan (for planning)
- Does NOT update task status directly

### Task_Manager
- **ONLY agent that updates task status**
- Performs all status transitions
- Manages Beads integration
- Updates beads metadata
- Validates transition validity

### plan agent
- Does NOT update task status
- Expects task to be in DOING status (updated by Task_Manager)
- Creates implementation plan only

### build agent
- Does NOT update task status directly
- May report blockers to Task_Manager
- Implements plan according to task requirements

### Code_Reviewer
- Does NOT update task status
- Reviews code and issues APPROVE/REJECT
- User decides next action based on review

### Repo_Manager
- Does NOT update task status
- Generates commit commands only
- User executes commits manually

---

## Common Patterns

### Pattern 1: Normal Task Flow
```
TODO (task created)
  ↓ (user: /plan T-XXXX)
DOING (Task_Manager updates, plan creates plan)
  ↓ (user: /build T-XXXX)
DOING (build implements, user reviews)
  ↓ (user confirms complete)
DONE (Task_Manager finalizes)
```

### Pattern 2: Task with Blocker
```
TODO
  ↓ (user: /plan)
DOING
  ↓ (build encounters issue)
BLOCKED (Task_Manager updates with blocker details)
  ↓ (user resolves blocker)
DOING (Task_Manager unblocks)
  ↓ (continues normally)
DONE
```

### Pattern 3: Task Skipped Early
```
TODO
  ↓ (user decides not to implement)
SKIP (Task_Manager updates with reason)
```

### Pattern 4: Task Abandoned During Work
```
TODO
  ↓ (user: /plan)
DOING
  ↓ (user decides to abandon)
SKIP (Task_Manager updates with reason, closes Beads)
```

---

## Error Prevention

### Preventing Invalid Transitions
Task_Manager MUST validate transition before applying:
```python
def validate_transition(current_status, new_status):
    if current_status == "DONE":
        return False, "DONE is final state"
    if current_status == "SKIP":
        return False, "SKIP is final state"
    if current_status == "TODO" and new_status == "DONE":
        return False, "Must go through DOING"
    return True, "Valid transition"
```

### Handling Transition Failures
If Task_Manager attempts invalid transition:
1. Reject transition with error message
2. Explain why transition is invalid
3. Suggest correct transition path
4. Do NOT modify task status

---

## Best Practices

1. **One task in DOING at a time** (focus)
   - Avoid multiple DOING tasks unless explicitly parallel work

2. **BLOCKED requires reason** (in notes)
   - Always explain what is blocking
   - Always specify what is needed to unblock

3. **SKIP requires reason** (in notes)
   - Always explain why task is skipped
   - Document decision for future reference

4. **DONE requires validation**
   - Ensure acceptance criteria met
   - Code reviewed and approved
   - Tests pass (if applicable)

5. **Keep history in beads metadata**
   - Never delete beads metadata
   - Provides audit trail
   - Enables incident recovery

---

## Troubleshooting

### Issue: Task stuck in DOING
**Symptoms**: Task in DOING for long time, no progress
**Solutions**:
- Review task notes for context
- Check if task should be BLOCKED (add blocker)
- Check if task should be SKIP (abandon work)
- Verify plan exists: `./plans/plan-T-XXXX.md`

### Issue: Task incorrectly marked DONE
**Symptoms**: Task is DONE but work incomplete
**Solutions**:
- Cannot reopen DONE task (final state)
- Create new task referencing original: "Complete remaining work from T-XXXX"
- Document what was missed in new task notes

### Issue: Beads out of sync with tasks.yaml
**Symptoms**: Beads issue exists but task.beads.mirrored = false
**Solutions**:
- tasks.yaml is authoritative
- Manually update task.beads metadata or close orphaned Beads issue
- Use Task_Manager to resync if needed

---

## Summary

**Key Points**:
- Task_Manager is the ONLY agent that updates task status
- TODO → DOING happens before planning (Task_Manager, then plan)
- DONE and SKIP are final states (cannot reopen)
- Beads mirrors DOING/BLOCKED tasks only
- All transitions are validated before applying
- Invalid transitions are rejected with clear error messages