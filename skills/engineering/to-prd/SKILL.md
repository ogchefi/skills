---
name: to-prd
description: Turn the current conversation context into a local Markdown PRD with built-in task tracking. Use when user wants to create a PRD from the current context.
---

# To PRD

Create a PRD from the current conversation context and codebase understanding. Do NOT interview the user from scratch; synthesize what you already know and ask only for missing details that block writing a useful PRD.

The PRD itself is the default backlog artifact. Create it as Markdown at `.scratch/<feature-slug>/PRD.md`, where `<feature-slug>` is a short kebab-case name for the feature or change. Create the feature directory if needed. Ask before overwriting an existing PRD.

External issue trackers are optional integrations, not the default output. If the user explicitly asks to publish elsewhere, follow the project issue-tracker docs if they exist.

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout the PRD, and respect any ADRs in the area you're touching.

2. Sketch out the major modules you will need to build or modify to complete the implementation. Actively look for opportunities to extract deep modules that can be tested in isolation.

A deep module (as opposed to a shallow module) is one which encapsulates a lot of functionality in a simple, testable interface which rarely changes.

Check with the user that these modules match their expectations. Check with the user which modules they want tests written for. Keep this confirmation short; do not restart discovery from scratch.

3. Write the PRD using the template below, then save it to `.scratch/<feature-slug>/PRD.md`.

4. Report the created PRD path and summarize the initial task count.

## Task Tracking Rules

The `## Tasks` table is the canonical task surface for PRDs created by this skill.

- Tasks should be vertical slices: each task delivers verifiable end-to-end behavior, not just work in one layer.
- New tasks start with status `Not started`.
- Allowed statuses are exactly: `Not started`, `In progress`, `Completed`, `Needs update`, `Pending removal`.
- Use stable task IDs in the form `Task 1`, `Task 2`, `Task 3`.
- Dependencies reference task IDs, or `None`.
- `Done when` describes externally verifiable completion criteria.
- `Notes` contains current-state context only. Do not use it as a changelog.
- Do not create a `## Temporary Update Notes` section in a new PRD. That section is only for `/update-prd` when existing task work needs reconciliation.

<prd-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Tasks

| ID | Task | Status | Done when | Dependencies | Notes |
| --- | --- | --- | --- | --- | --- |
| Task 1 | First vertical slice to implement. | Not started | Externally verifiable completion condition. | None | Current-state context, if any. |

## Out of Scope

A description of the things that are out of scope for this PRD.

## Further Notes

Any further notes about the feature.

</prd-template>
