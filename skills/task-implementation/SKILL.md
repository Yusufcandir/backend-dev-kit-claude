---
name: task-implementation
description: Implements an analyzed task or roadmap in code, reports problems, gaps and ambiguities found while coding, and runs a completeness check against the roadmap when coding is done. Use when the user says "implement the roadmap", "code this task" or "build this".
---

# Task Implementation + Issue Reporting + Completeness Check

Input: the roadmap produced by the `task-analysis` skill (or an equivalent analysis document).
If there is no roadmap, first run the `task-analysis` skill to produce one, then continue with this skill.

This skill has three phases. Run them in order.

---

## Phase 1 — Preparation

1. Read the roadmap; extract the implementation steps, acceptance criteria (AC) and test plan.
2. Study the codebase: architecture, naming, error handling, logging, DI, testing and migration conventions.
   **New code must mimic the style of the existing code.**
3. Find the build and test commands (README, Makefile, package.json scripts, CI files).
4. Record the current test status (do tests pass before you change anything?). Report tests that were already failing separately.
5. Create a **Tracking List** (use the TodoWrite/Task tool if available): one item per roadmap step.

## Phase 2 — Implementation

For each roadmap step:

1. Implement the step's to-do items. You can delegate the coding to the `backend-coder` agent
   (well suited to running large or independent steps in parallel).
2. Write the step's tests (match them to the T-x IDs in the roadmap).
3. Run the build and the relevant tests. If they're red, fix them; if you can't, add it to the Issue Log.
4. Mark the step as done in the Tracking List.

### Issue Log — when to report while coding

When any of the following happens, give the user a short note **right away** and add it to the Issue Log.
If it doesn't block the work, continue with a reasonable solution; if it does, skip that step and continue with the next independent one.

| Type | Example |
|---|---|
| `MISSING-REQUIREMENT` | The roadmap doesn't say what to return on error |
| `CONFLICT` | The roadmap says X, but the existing code/schema doesn't allow Y |
| `TECHNICAL-BLOCKER` | A required library/service/access is missing, a migration conflicts |
| `RISK` | The change breaks backward compatibility, risk of a performance problem |
| `OUT-OF-SCOPE` | Working correctly needs a change outside the roadmap |
| `TECH-DEBT` | A problem noticed in the existing code that this task doesn't fix |

Entry format:
```
[TYPE] <short title>
- Where: <file:line or roadmap step>
- Problem: ...
- Impact: ...
- Action taken / Suggestion: ...
- Blocking: Yes/No
```

### Coding rules

- Only make changes within the roadmap's scope; no unrelated refactoring (report it as `OUT-OF-SCOPE` if needed).
- Never hardcode secrets (passwords, tokens, keys); use the existing configuration mechanism.
- Add input validation, authorization, error handling and logging following the existing patterns.
- Make database changes through migrations, and make them reversible (down/rollback).
- Never skip or disable tests, or weaken assertions "to make them pass".

## Phase 3 — Completeness Check

When coding is done, check systematically against the roadmap:

1. **Steps:** Was every implementation step and sub-item done?
2. **Requirement traceability:** Each FR/NFR → which file satisfies it → which test verifies it?
3. **Acceptance criteria:** Evidence for each AC (test name, command output or code reference).
4. **Test plan:** Was every T-x in the roadmap written, and does it pass?
5. **Edge cases and security checklist:** Was every item in sections 8 and 9 of the roadmap handled?
6. **Cleanup:** No leftover TODO/FIXME, debug logs, commented-out code or unused imports?
7. Run the **full build + entire test suite** one last time; put the result in the report.

If something is missing, complete it and repeat the check. If it can't be completed, list it in the report as `MISSING`.

## Final report (template)

```markdown
# Implementation Report: <Task name>

## Status: ✅ Complete | ⚠️ Partially complete | ❌ Blocked

## Changes made
| File | Change | Roadmap step |
|---|---|---|

## Completeness matrix
| Requirement / AC | Where satisfied | Test | Status (✅/⚠️/❌) |
|---|---|---|---|

## Test results
- Command: `...`
- Result: X passed / Y failed / Z skipped
- Previously failing tests: ...

## Issue Log
<All issues logged in Phase 2, blockers first>

## Still missing
- ... (or "None")

## Next step
Run the `task-review` skill for code review and testing.
```
