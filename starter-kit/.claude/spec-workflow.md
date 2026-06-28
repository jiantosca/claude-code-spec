# Spec-driven development workflow

> Imported into `CLAUDE.md` via `@.claude/spec-workflow.md`. This file holds the
> spec workflow rules so they stay separate from your project-specific guidance
> and can be updated (or centralized) on their own.

## Right-sizing (read first)

Match ceremony to the task — don't spec everything:
- **One-sentence diff** (typo, rename, tiny bug): just do it, no spec.
- **Uncertain / multi-file / unfamiliar, one sitting**: use plan mode (shift+tab).
- **Multi-step, want a durable committed checklist**: `/spec-tasks <feature> <description>` (standalone), then `/spec-impl`.
- **Big feature, shared design, multiple sessions/people**: full spec flow below.

Native Tasks (`TaskCreate`/`TaskUpdate`/…) track per-machine live progress and
do NOT travel in git. `tasks.md` is the committed, team-reviewable plan. Use both.

## Spec-driven development

Features are specified before implementation, in `.claude/specs/<feature>/` as
three files produced in order:

1. `requirements.md` — user stories + acceptance criteria in EARS notation
2. `design.md` — architecture, data models, file structure plan
3. `tasks.md` — discrete, dependency-annotated, checkbox task list

Build a spec with these commands, approving each phase before the next:

- `/spec-requirements <feature> <description>`
- `/spec-design <feature>`
- `/spec-tasks <feature>` (adaptive: also works standalone with a description if
  there's no requirements/design)

Each phase follows the same rhythm: **clarify open questions with the user before
drafting** (ask the questions that change what gets written — don't draft on
silent assumptions), produce the artifact, iterate to explicit approval, then
point at the next command. Never skip ahead to the next phase unprompted.

Implement it:

- `/spec-impl <feature> <task#>` — one task at a time, stops for check-in
- `/spec-impl <feature> all` — all tasks, respecting dependencies

If requirements change after design/tasks exist, run `/spec-sync <feature>`.

**Rule:** never implement a feature that doesn't have an approved `tasks.md`.
