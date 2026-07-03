# Spec-Driven Development starter kit for Claude Code

A Kiro-style spec workflow built from Claude Code's native primitives. Drop it
into any repo and you get `/spec-*` slash commands that produce
`requirements.md` → `design.md` → `tasks.md` and implement them task-by-task.

## Install

Copy these into the root of your repo:

```
.claude/commands/        # the slash commands
.claude/spec-workflow.md # the workflow rules Claude follows
.claude/settings.json    # optional hooks: half-install warning + task reminder (see below)
CLAUDE.md                # one line that @imports spec-workflow.md
```

The spec workflow lives in `.claude/spec-workflow.md`, and `CLAUDE.md` pulls it in
with a single `@.claude/spec-workflow.md` import. **Already have a `CLAUDE.md`?**
Don't overwrite it — just add that one import line and drop in `spec-workflow.md`.
Keep your own project rules in CLAUDE.md's "Project-specific guidance" section.

Specs themselves are written to a top-level `specs/<feature>/` directory —
deliberately *outside* `.claude/`, so Claude can edit them (e.g. ticking task
checkboxes during `/spec-impl`) without tripping the permission prompt that
guards `.claude/` config. `specs/example-feature/` here is a worked example (a
dark-mode toggle) so you can see the target output; it lives outside `.claude/`,
so copying `.claude/` into your repo won't drag it along.

Already use Kiro at work? See `MIGRATING-FROM-KIRO.md` for a side-by-side and the
behavioral differences to watch for.

Wondering why this hand-written kit instead of an off-the-shelf tool like
`cc-sdd`? See "Why roll our own vs. adopt cc-sdd" in
`../kiro-spec-driven-dev-in-claude-code.md`.

## Sharing across projects & your team

Copying works, but you don't have to. The `/spec-*` commands encode a *process*
that's identical from project to project — so centralize them once instead of
forking a copy into every repo. Two things travel separately:

- the **commands** (`spec-*.md`) — shareable, should *not* be customized per app
- the **CLAUDE.md guidance** — per-project, teams will customize it

Pick the mechanism that matches your goal:

| Goal | Mechanism | How |
|---|---|---|
| Commands available in **all your own** projects, no copying | **User-level commands** | Put `spec-*.md` in `~/.claude/commands/` instead of each repo. Per-user, per-machine — not git-shared. |
| Dedupe the **guidance text** across repos | **CLAUDE.md `@import`** | The kit already splits the workflow into `.claude/spec-workflow.md` and imports it. To share one copy across repos, move it to `~/.claude/spec-workflow.md` and point each project's import at `@~/.claude/spec-workflow.md`, customizing only the project-specific lines in each CLAUDE.md. |
| Commands shared with a **team across many repos**, versioned | **Plugin** ⭐ | Package the commands (and optionally the guidance as a skill) into a git repo with a `.claude-plugin/plugin.json` manifest. Install via `/plugin marketplace add <org/repo>` then `/plugin install`. Versioned and auto-namespaced (`/spec-kit:spec-tasks`), so improvements roll out centrally with no fork drift. |
| Commands shared within a **single repo** | **Project-level (the default Install above)** | Commit `.claude/commands/` — the method travels with the repo. Simplest; start here. |

Rule of thumb: start with the project-level copy (it's zero-setup and the whole
team gets it via git). Reach for `@import` once you're maintaining the guidance in
more than one place, and graduate to a **plugin** when you want one versioned
source of truth for the commands across many repos.

Note: only the *commands and guidance* centralize this way. The spec artifacts
themselves (`specs/<feature>/`) are intentionally per-project and
committed to that repo — same as Kiro's `.kiro/specs/`.

## Workflow

```
/spec-requirements <feature> <description>   # writes requirements.md, stops
/spec-design <feature>                        # writes design.md, stops
/spec-tasks <feature> [description]           # writes tasks.md, stops
/spec-impl <feature> 1                        # implement task 1, stop (check-in mode)
/spec-impl <feature> all                      # implement all, dependency-ordered
/spec-sync <feature>                          # re-reconcile after editing requirements
```

Approve each phase before running the next — that's the Kiro gating model.
Tip: use plan mode (shift+tab) during the requirements/design phases so Claude
proposes without touching files.

`/spec-tasks` is adaptive: if `requirements.md` + `design.md` exist it builds
tasks from them; if they don't, pass a description and it generates a standalone
`tasks.md` (the lightweight, no-full-spec path). For larger features, run
`/spec-impl` in a *fresh* session so it executes against the written spec with
clean context (an Anthropic-recommended practice).

## Right-sizing: when to use what

Don't run every change through a spec — that's the most common spec-driven
anti-pattern, and Anthropic's own guidance says to skip planning for small work.
Match the ceremony to the task:

| Tier | When | What to do |
|---|---|---|
| **1. Just do it** | You could describe the diff in one sentence (typo, log line, rename, tiny bug) | Skip all of this. Ask Claude directly. |
| **2. Plan mode** | Uncertain approach, multi-file, or unfamiliar code — but a one-sitting job | `shift+tab` into plan mode; no files needed |
| **3. tasks.md only** | A multi-step job you want as a durable, resumable, committed checklist | `/spec-tasks <feature> <description>` then `/spec-impl` |
| **4. Full spec** | Big refactor/feature; multiple sessions/people/agents; design must be shared and reviewed | `/spec-requirements` → `/spec-design` → `/spec-tasks` → `/spec-impl` |

Rule of thumb: a spec earns its cost when multiple sessions/developers/agents
touch the same feature, outputs must match a shared design, or the cost of drift
exceeds the cost of writing it down. Otherwise, go lighter.

## tasks.md vs. native Tasks

Claude Code has a built-in **Tasks** primitive (the `TaskCreate`/`TaskUpdate`/…
tools, default since v2.1.142). It persists across sessions and subagents on the
local filesystem (`~/.claude/tasks`). That's great for *your* live work state —
but it is **machine-local and never uploaded**, so it does not travel in git and
your teammates never see it.

This kit's `tasks.md` is the complement: a **committed, reviewable, team-shared**
checklist that lives in the repo. Use native Tasks for ephemeral per-machine
progress; use `tasks.md` when the plan itself is an artifact the team should
review and keep in sync with the code. They coexist fine — Claude can track
in-flight progress with native Tasks while `tasks.md` stays the source of truth.

## The hooks

`.claude/settings.json` ships two **non-blocking** hooks (delete either entry, or
the whole `"hooks"` block, to remove them):

- a **`SessionStart`** check that shows a visible warning (via a `systemMessage`,
  so it appears in the terminal and the IDE extensions alike) if
  `.claude/spec-workflow.md` exists but your `CLAUDE.md` doesn't `@import` it —
  i.e. a half-install where the `/spec-*` commands work but the workflow
  guardrails and right-sizing guidance never load.
- a **`Stop`** reminder that nudges you to tick task checkboxes in
  `specs/<feature>/tasks.md` after implementing.

The `SessionStart` check assumes the default `.claude/spec-workflow.md` path. If
you centralize the workflow elsewhere (e.g. `@~/.claude/spec-workflow.md` — see
"Sharing across projects & your team"), update the path in the hook to match, or
just delete that `SessionStart` entry — it only guards the in-repo layout.

To make it **strict** instead (Kiro-like gating that blocks source edits until an
approved tasks.md exists), replace the `Stop` hook with a `PreToolUse` hook on
`Edit|Write` that checks for the tasks file and exits non-zero to block. Ask
Claude "make the spec hook blocking" and it can wire it for you.

## Why this mirrors Kiro

| Kiro | Here |
|---|---|
| Requirements/Design/Tasks phase buttons | `/spec-requirements`, `/spec-design`, `/spec-tasks` |
| Per-phase approval gates | Each command stops and waits for you |
| Run task / Run all Tasks | `/spec-impl <n>` vs `/spec-impl all` |
| Steering files | `CLAUDE.md` + `@import`ed `.claude/spec-workflow.md` |
| Sync | `/spec-sync` |
| Specs in the repo | `specs/<feature>/`, committed to git |

Everything is plain markdown + git — no special tooling, and the *method* itself
travels in the repo so the whole team stays in sync.
