---
name: update-prd
description: Update an existing PRD from the current conversation context while keeping the PRD as a clean source of truth. Use when user wants to revise, reconcile, reopen, remove, or clean up PRD requirements, tasks, acceptance criteria, implementation decisions, testing decisions, or temporary PRD update notes.
---

# Update PRD

Update an existing PRD from current context or explicitly specified tasks. This complements `/to-prd`: `/to-prd` creates a local Markdown PRD with built-in task tracking, while this skill revises an existing PRD without turning it into a changelog.

Do not interview from scratch. Explore the repo and the PRD enough to understand the current state, then ask only when the target PRD, affected task, or requested lifecycle change is ambiguous.

The PRD itself is the default backlog artifact. If the PRD lives in an external issue tracker because the user or project explicitly chose that, preserve that backend and follow the project issue-tracker docs.

## Process

1. Find the target PRD:
   - Prefer explicit paths, issue links, or PRD titles from the user.
   - If no target is explicit, search likely PRD locations, especially `.scratch/*/PRD.md`, then issue tracker context if configured.
   - Ask before updating if multiple plausible PRDs remain.
   - Ask before updating more than one PRD.
   - Ask before touching any file outside the active repo.
   - If the PRD lives in an issue tracker, update the existing issue body. Do not create a replacement issue.

2. Read the PRD and preserve its section semantics:
   - Put requirement changes in `Solution`, `User Stories`, acceptance criteria, or equivalent requirement sections.
   - Put engineering choices in `Implementation Decisions`.
   - Put test expectations in `Testing Decisions`.
   - Put exclusions in `Out of Scope`.
   - Keep `Further Notes` for canonical assumptions, risks, dependencies, and follow-ups. Do not use it for change history.

3. Rewrite canonical PRD content cleanly:
   - State the current truth directly.
   - Do not inline old wording, before/after notes, revision notes, or historical commentary.
   - Remove stale proof of completion when reopening changed work.
   - Preserve the PRD's existing structure and tone unless the structure itself is causing confusion.

4. Handle PRD task surfaces carefully:
   - Treat PRD-internal tasks, acceptance checklists, task status lists, and status tables as the authoritative task surface by default.
   - Prefer the canonical `## Tasks` table for PRDs created by `/to-prd`.
   - If a PRD has no task surface and the user asks for task tracking, add the canonical `## Tasks` table.
   - Touch separate implementation plans only when the user explicitly names them.
   - Recognize common Markdown styles: `- [ ]`, `- [x]`, numbered task lists, acceptance checklist items, and task/status tables.
   - Preserve the local task style for legacy PRDs unless the user asks to normalize it.

## Canonical Task Table

For PRDs created by `/to-prd`, use this task table shape exactly:

```md
## Tasks

| ID | Task | Status | Done when | Dependencies | Notes |
| --- | --- | --- | --- | --- | --- |
| Task 1 | Example vertical slice. | Not started | Externally verifiable completion condition. | None | Current-state context, if any. |
```

Rules:

- Allowed statuses are exactly: `Not started`, `In progress`, `Completed`, `Needs update`, `Pending removal`.
- Tasks should remain vertical slices: each task delivers verifiable end-to-end behavior, not just work in one layer.
- Preserve stable task IDs. Do not renumber tasks unless the user explicitly asks.
- New task IDs use the next unused `Task N`.
- Dependencies reference task IDs, or `None`.
- `Done when` describes externally verifiable completion criteria.
- `Notes` contains current-state context only. Do not use it as a changelog.

## Task Lifecycle Rules

When adding a new task:

- Add it cleanly as incomplete.
- For the canonical task table, append it with the next unused task ID and status `Not started`.
- Do not add a temporary update note.

When editing an existing task:

- Mark it incomplete.
- Use `Needs update` for status-table or status-list PRDs.
- For the canonical task table, keep the same task ID, rewrite the current task fields, and set status `Needs update`.
- Remove or rewrite stale nested completion details so they express current requirements, not past evidence.
- Add one temporary update note for that task.

When starting work on a task:

- Set status `In progress` only when explicit user instruction, named task results, or current-chat implementation evidence says work has started.
- Do not add a temporary update note just because work started.

When removing a task:

- Do not delete it immediately if related implementation may still exist.
- Mark it incomplete and `Pending removal`.
- For the canonical task table, keep the task row, set status `Pending removal`, and describe the cleanup condition in `Notes` or the temporary note.
- Add one temporary update note with the cleanup condition.
- Delete the task only after explicit current-chat evidence, named task results, or user instruction says the related implementation cleanup is complete.

When cleanup is complete:

- Remove the matching temporary note.
- Mark edited tasks complete only when explicit user instruction, named task results, or current-chat implementation evidence says the updated work is done.
- Delete pending-removal tasks only when explicit evidence says implementation cleanup is done.
- Remove the temporary section entirely when no notes remain.

Do not infer completion from code shape alone. Code inspection can inform the update, but PRD completion is a project truth claim.

Preserve issue tracker metadata unless the user asks to change it. Do not change triage roles, labels, assignees, milestones, or issue state just because the PRD body changed.

## Temporary Update Notes

Use a bottom section named exactly:

```md
## Temporary Update Notes
```

This section is allowed inside the PRD, but it must stay separate from canonical PRD sections. It exists only to highlight reconciliation work after an existing task is edited or marked for removal.

Use this format:

```md
## Temporary Update Notes

Remove this section after all listed PRD updates have been reconciled in implementation.

| Task | Type | Summary | Cleanup condition |
| --- | --- | --- | --- |
| Task 3: Example | Needs update | Requirement-level summary of what changed. | Remove after the updated behavior is implemented and validated. |
```

Rules:

- Use one row per affected existing task.
- Keep summaries concise and requirement-level.
- Do not copy old task text.
- Do not include before/after diffs.
- Do not preserve old completion evidence.
- Merge with an existing `Temporary Update Notes` section instead of creating duplicates.

## Ambiguity Rules

Ask before changing task status when you cannot confidently map a requested change to an existing task.

If the PRD target is clear but task mapping is ambiguous, you may still cleanly update non-task PRD sections. Do not reopen, complete, or remove task entries until the task mapping is clear.

## Final Response

Report:

- Which PRD was updated.
- Which tasks were added, moved to `In progress`, marked `Needs update`, marked `Pending removal`, completed, or deleted.
- Whether `Temporary Update Notes` were added, changed, or cleaned up.
- Any ambiguity that still needs user confirmation.
