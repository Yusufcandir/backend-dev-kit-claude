---
name: task-analysis
description: Turns a raw task (a Jira/issue description, a user request, a bug report) into something ready to code and directly produces a step-by-step development roadmap. Use when the user shares a task, issue or feature request, or says "analyze this", "give me a roadmap" or "how should we build this".
---

# Task Analysis → Development Roadmap

This skill has exactly one output: **a Roadmap document that follows the template below exactly.**
Do NOT wait for extra information, approval or clarification from the user. However incomplete the task is,
produce the roadmap; write the unknowns into the "Assumptions" and "Open Questions" sections and keep going.

## Core rules

1. **Never stop to ask questions.** When something is ambiguous, make the most reasonable assumption, mark it `[ASSUMPTION]`, and continue the roadmap.
2. **If there is a codebase, read it first.** If a project exists in the working directory, inspect: the directory layout, the language/framework,
   package files (package.json, pom.xml, build.gradle, go.mod, requirements.txt, pyproject.toml, *.csproj, etc.),
   the existing layering (controller/service/repository), the test folder, the migration folder, and configuration.
   Use the existing code's file paths and naming in the roadmap. If there is no codebase, write it stack-agnostic
   and add the assumed stack to "Assumptions".
3. **Stay stack-agnostic.** Don't tie the roadmap to one language; use the terms of whatever the project uses.
4. **Every step must be actionable on its own.** A developer (or another AI) should be able to do the work by reading only that step.
5. **Measurable acceptance criteria.** No vague phrases like "should work"; write testable ones such as "request X returns 409 when Y".
6. If research is needed (an unfamiliar library, an external API, a standard), you may use the `researcher` agent; otherwise use your own knowledge.

## Analysis process (do this internally; put only the result in the output)

1. **Understand:** What is the business goal of the task? Who will use it? What problem does it solve?
2. **Break down:** Functional requirements, non-functional requirements (performance, security, scalability,
   observability), constraints.
3. **Impact area:** Which modules, endpoints, tables, queues, services and configuration are affected?
4. **Data model and contracts:** New or changed entities, tables, fields, indexes, DTOs, API contracts, event schemas.
5. **Edge cases:** Empty/null input, unauthorized access, concurrency, idempotency, large data, timeouts,
   external service failure, backward compatibility.
6. **Risks:** Technical risks, data loss/migration risk, performance risk, security risk.
7. **Split:** Break the work into small, testable steps in dependency order (ideally a few hours of work each).

## Output template (produce it with exactly these headings)

```markdown
# Roadmap: <Short task name>

## 1. Summary
<2-4 sentences: what will be built, why, and what the outcome will be>

## 2. Scope
**In scope:**
- ...
**Out of scope:**
- ...

## 3. Requirements
### Functional
- FR-1: ...
- FR-2: ...
### Non-functional
- NFR-1: (performance / security / logging / scalability) ...

## 4. Assumptions
- [ASSUMPTION] ...

## 5. Technical Design
### Affected components
| Component / File | Change type (new/modified/deleted) | Description |
|---|---|---|
### Data model
<new or changed tables, fields, indexes, migrations; otherwise "No changes">
### API / Contracts
| Method | Path / Event | Request | Response | Error codes |
|---|---|---|---|---|
### Flow
<step-by-step request/business flow; a short ordered list if needed>

## 6. Implementation Steps
### Step 1: <title>
- **Goal:** ...
- **To do:**
  - [ ] ...
- **Files:** `path/to/file`
- **Depends on:** None | Step N
- **Done when:** ...
### Step 2: ...
(… continue in dependency order)

## 7. Test Plan
| ID | Type (unit/integration/e2e) | Scenario | Expected result |
|---|---|---|---|
| T-1 | unit | ... | ... |
(At least one positive and one negative test per FR; one test per edge case)

## 8. Edge Cases and Error Handling
- ...

## 9. Security Checklist
- [ ] Authentication / authorization
- [ ] Input validation
- [ ] Sensitive data (logging, masking, encryption)
- [ ] (task-specific items)

## 10. Risks
| Risk | Impact (Low/Medium/High) | Mitigation |
|---|---|---|

## 11. Acceptance Criteria
- [ ] AC-1: ...
- [ ] AC-2: ...

## 12. Open Questions
- ... (work can proceed without an answer; the chosen assumption is in section 4)

## 13. Effort Estimate
| Step | Estimate |
|---|---|
| Total | ... |
```

## Quality check (before giving the output)

- Is every FR linked to at least one implementation step and one test?
- Does every step have a "Done when" criterion?
- Are the steps in dependency order?
- Are the acceptance criteria testable?
- Is no section empty? (If a section doesn't apply, write "Not applicable — <reason>")

Once the roadmap is complete, the `task-implementation` skill takes this document as its input to start coding.
If the user asked for a file, also save the roadmap as `docs/roadmap/<task-name>.md`.
