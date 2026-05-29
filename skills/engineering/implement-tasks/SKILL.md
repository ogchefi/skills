---
name: implement-tasks
description: Execute tracked PRD tasks or generic Markdown task plans end-to-end with subagent implementation, two review-and-patch passes, blocker tracking, plan status updates, validation, and per-task commits. Use when the user invokes /implement-tasks, says "Implement tasks", asks Codex to execute PRD tasks, or asks Codex to implement a markdown plan or tracked task list.
---

# Implement Tasks

Implement a provided task plan one task at a time. Keep momentum, but do not hide blockers that can make later tasks difficult or impossible.

## Preconditions

- User must provide a clear plan, issue body, checklist, or `.md` file path. If not, ask for it and stop.
- Before implementation, identify the task source mode:
  - PRD task table mode: a Markdown PRD with a canonical `## Tasks` table using columns `ID`, `Task`, `Status`, `Done when`, `Dependencies`, and `Notes`.
  - Generic Markdown mode: another issue body, checklist, numbered plan, status list, or status table with clear task boundaries and tracking state.
- If the plan has no tracked tasks, ask: "Plan has no tracked tasks. Add them automatically with a subagent, or abort?"
- If user chooses automatic task creation, spawn one planning subagent and omit `model` so it inherits the parent model unless user requested another model. Ask it to add the smallest useful tracked task list to the plan. It must not ask to enable caveman.
- Do not start implementation until tasks can be selected and marked in progress/done.

## PRD Task Table Mode

Use this mode by default for PRDs created or maintained by `/to-prd` or `/update-prd`.

- Treat the PRD `## Tasks` table as the authoritative execution queue.
- Preserve stable task IDs exactly. Do not renumber tasks.
- Use the existing PRD status vocabulary exactly: `Not started`, `In progress`, `Completed`, `Needs update`, `Pending removal`.
- Select executable tasks in dependency order:
  - `Not started`: normal implementation work.
  - `Needs update`: reconciliation work for changed requirements; implement the current PRD truth, not stale completion notes.
  - `Pending removal`: cleanup/removal work; only delete the task row after explicit evidence that related implementation cleanup is complete.
  - `In progress`: continue only if it is the next safe task and the repo state matches that task.
  - `Completed`: skip unless the user explicitly asks to reopen it.
- Set the selected task to `In progress` before spawning the implementer.
- After implementation, validation, staging, and commit succeed, set normal tasks to `Completed`.
- For `Pending removal` tasks, remove the row only when cleanup is implemented and validated; otherwise keep it `Pending removal` and stop or report the blocker.
- If a `## Temporary Update Notes` section exists, use it as reconciliation context. Remove only the rows whose cleanup condition was satisfied by the completed task. Remove the section entirely only when no temporary notes remain.
- Do not change requirement sections, add new PRD tasks, or infer PRD completion beyond the task being executed unless a blocker or non-blocking planning follow-up requires it.

## Generic Markdown Mode

Use this mode for non-PRD plans, issue bodies, legacy checklists, and ad hoc tracked Markdown.

- Preserve the plan's existing task style and status vocabulary.
- Treat unchecked checklist items, incomplete numbered items, explicit todo rows, and equivalent incomplete statuses as executable tasks.
- Mark the selected task in progress when the format supports it. If it does not, keep local tracking notes in the running book and avoid inventing a new status system.
- Mark the selected task done only after implementation, validation, staging, and commit succeed.
- If task boundaries or status meaning are ambiguous, ask before implementing.

## Parent Responsibilities

- Own orchestration, task selection, blocker decisions, plan tracking, validation, staging, and commits.
- Only the parent may stage, commit, amend, push, or open PRs. Subagents must leave all changes unstaged and uncommitted unless the user explicitly overrides this skill instruction.
- Respect repository instructions and dirty worktrees. Do not revert unrelated edits.
- Use subagents only for task implementation and review passes. When spawning, omit `model` by default so subagents use the same model as the parent; set a model only if the user explicitly requested one.
- Tell every subagent: it is not alone in the codebase, must not revert others' edits, must work only inside its assigned scope, must not stage or commit changes, and must not ask to enable caveman.
- Keep a running book of:
  - active task source mode
  - current task ID or label
  - hard blockers
  - non-blocking decisions
  - planning follow-up tasks
  - validations run or skipped

## Per-Task Loop

For each pending implementation task:

1. Mark the task in progress in the plan when the task source supports it; otherwise record it in local tracking notes.
2. Spawn an implementer subagent for only this task. Require it to edit files directly, run focused checks when practical, avoid staging or committing, and report changed paths plus any user decisions needed.
3. When implementer finishes, inspect status and changed files enough to understand scope.
4. Spawn reviewer subagent pass 1. Require it to review the implementer's changes, patch actionable findings directly in files only, avoid staging or committing, run focused checks when practical, and report changed paths plus blockers or decisions.
5. Inspect status and changes again.
6. Spawn reviewer subagent pass 2 with the same review-and-patch mandate: file edits are allowed, but staging, committing, amending, pushing, and PR actions are reserved for the parent.
7. Classify remaining decisions:
   - Hard blocker: stop before committing if it can make remaining task implementation difficult or impossible.
   - Non-blocking decision: add a planning task or note after all implementation tasks are done. Continue.
8. Run required validation for the touched app/module. For TypeScript or JavaScript changes, run type and lint checks required by repo instructions. Fix certain issues; ask only when the fix is uncertain.
9. Mark the task complete in the plan, stage only relevant files, and commit the task. Use repository commit message conventions.
10. Move to the next pending task.

## Stop Conditions

Stop and report clearly when:

- There is no usable plan or task tracking and user did not authorize automatic task creation.
- PRD task table dependencies make the next task unsafe to start.
- A hard blocker remains after the implementer and both reviewer passes.
- Required validation fails and the fix is uncertain.
- Repository state makes a clean per-task commit unsafe without user direction.

## Final Report

Include:

- Task source mode used.
- Tasks completed and commits created.
- Tasks left pending, if any.
- Hard blockers, if stopped.
- Planning tasks added for non-blocking decisions.
- Temporary update notes removed or left pending, if any.
- Validation commands and results.
