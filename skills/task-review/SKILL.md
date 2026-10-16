---
name: task-review
description: Checks an implemented task, runs the tests and writes missing ones, does a detailed code review including security, and gives prioritized feedback. Use when the user says "review this", "check the code", "test it", "give feedback" or "review the PR".
---

# Code Check + Testing + Code Review + Feedback

Input: the change to review. Determine the source in this order:
1. If the user gave a PR / branch / commit / file, use that.
2. Otherwise, the changes in the working directory: `git diff` + `git diff --staged`; if empty, the diff against the main branch
   (`git diff $(git merge-base HEAD main)` — use master/develop if there is no main).
3. If they exist, also read the roadmap (`task-analysis` output) and the implementation report (`task-implementation` output),
   and review against them.

Don't wait for extra information from the user; determine the scope yourself using the order above and state it in the report.

## Step 1 — Understand the context
- What is the change for? (roadmap, commit messages, PR description)
- Read the changed files and the places that call them or that they call. Look not only at the diff but at the code it affects.

## Step 2 — Verification (can run in parallel)
Start these agents **at the same time** and merge their results:
- `test-engineer`: runs the existing tests, measures the change's test coverage, writes and runs missing tests.
- `security-auditor`: reviews the change from a security perspective.

If agents aren't available, do this yourself: build, lint, all tests; write the missing critical tests.

## Step 3 — Code review checklist

**Correctness**
- Logic errors, off-by-one, null/empty values, wrong conditions, faulty error handling
- Concurrency: race conditions, transaction boundaries, locks, idempotency
- Failure cases: external service failure, timeouts, retries, partial failure
- Does it actually meet the roadmap's requirements and acceptance criteria?

**Design and maintainability**
- Fit with the existing architecture and patterns; layering violations (e.g. business logic in a controller)
- Duplicated code, unnecessary complexity, distribution of responsibilities
- Naming, readability, dead code, TODO/FIXME

**Data and API**
- Is the migration safe and reversible? Locking risk on large tables?
- Index needs, N+1 queries, fetching unnecessary data
- Is the API backward compatible? Are error codes and response formats consistent?

**Performance and operations**
- I/O inside loops, high memory use, missing pagination
- Log levels and content (sensitive data must not be logged), metrics/traces
- Configuration: hardcoded values, environment dependencies

**Tests**
- Do the tests verify behavior or implementation? Any meaningless assertions?
- Are negative scenarios and edge cases tested?

## Step 4 — Prioritize the findings

| Level | Meaning |
|---|---|
| 🔴 CRITICAL | Must not be merged: bug, data loss, security hole, broken test/build |
| 🟠 HIGH | Must be fixed before merge: missing requirement, important edge case, serious performance issue |
| 🟡 MEDIUM | Should be fixed: maintainability, readability, missing test |
| 🔵 LOW | Suggestion / style / small improvement |

Every finding must be concrete: file:line, what the problem is, why it matters, and how to fix it (with a code example when possible).
Mark findings you're unsure of as "unverified"; don't present guesses as facts.

## Report template

```markdown
# Code Review: <Task / PR name>

## Verdict: ✅ Approved | 🔁 Changes requested | ❌ Rejected
<1-2 sentence rationale>

## Summary
- Scope reviewed: <diff source, number of files>
- Findings: 🔴 x · 🟠 x · 🟡 x · 🔵 x

## Test results
- Build: ✅/❌
- Tests: X passed / Y failed (command: `...`)
- Newly added tests: ...
- Coverage note: ...

## Requirements check
| Requirement / AC | Status | Note |
|---|---|---|

## Findings
### 🔴 CRITICAL
#### C-1: <title>
- **Location:** `file:line`
- **Problem:** ...
- **Why it matters:** ...
- **Suggestion:** ...
### 🟠 HIGH
...
### 🟡 MEDIUM
...
### 🔵 LOW
...

## Security summary
<summary from the security-auditor result>

## What's good
- ...

## To do (in priority order)
- [ ] ...
```

If there are no findings, don't make any up; write "No findings" and base the verdict on that.
If the user wants, apply the fixes with the `backend-coder` agent and repeat the review.
