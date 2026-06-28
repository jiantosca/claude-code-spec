# Kiro Spec-Driven Development → Claude Code

> Research compiled 2026-06-27. Kiro mechanics verified against primary sources
> (mostly `kiro.dev` official docs) and the two leading community ports. Claude
> Code replication combines those sources with firsthand knowledge of Claude
> Code's mechanics. Source notes are inline in *(parentheses)*.

---

## 1. What Kiro actually does

The verified skeleton — details matter for replication:

- **Three sequential artifacts** per feature:
  - `requirements.md` — user stories + acceptance criteria in **EARS** notation
    (`WHEN [condition] THE SYSTEM SHALL [behavior]`)
  - `design.md` — architecture, sequence diagrams, data models,
    components/interactions
  - `tasks.md` — discrete, trackable tasks with descriptions, expected outcomes,
    and **dependencies between tasks**
  *(kiro.dev/docs/specs, /feature-specs)*

- **Gated phases in standard mode**: "The standard spec flow has three phases:
  requirements, design, tasks. You approve each one before the next begins."
  There's also a **Quick Plan mode** that generates all three without approval
  gates, and a **Design-First** variant (Design → Requirements → Tasks) alongside
  the default Requirements-First.
  *(kiro.dev/blog/faster-smarter-specs, /docs/specs/feature-specs/tech-design-first)*

- **Execution: one-at-a-time *or* all-at-once.** Originally one task at a time
  *by design* — "You had to check in with the agent after every task. That wasn't
  an oversight. It was intentional" (visibility/control). The newer **Run all
  Tasks** builds a dependency graph from `tasks.md`, groups independent tasks into
  **waves** that run concurrently, and validates as it goes.
  *(kiro.dev/blog/run-all-tasks, /docs/specs/best-practices)*

- **Version control is just plain git.** The research *refuted* the idea that Kiro
  has any special VC handling for spec files — the source-control docs cover only
  ordinary git (AI commit messages, conventional commits). The specs are just
  markdown in the repo; "sync" is a Kiro feature that regenerates downstream
  artifacts when you edit an upstream one, **not** a git operation.
  *(kiro.dev/docs/editor/source-control — verified absence)*

**Takeaway:** there is no magic to reproduce on the version-control side. Markdown
files in the repo + git is the whole story. The thing worth reproducing is the
*phase pipeline* and the *task-by-task execution loop*.

---

## 2. Concept → Claude Code primitive mapping

| Kiro concept | Claude Code equivalent |
|---|---|
| Spec phase buttons (Requirements/Design/Tasks) | **Slash commands** (`.claude/commands/*.md`) — one per phase |
| The opinionated methodology/system prompt | **CLAUDE.md** (loaded at session start, reloaded after compaction) + the command bodies |
| Approval gate between phases | **Plan mode** (`shift+tab`) and/or commands that stop and ask for confirmation |
| `requirements.md` / `design.md` / `tasks.md` | The same three files, written to a per-feature dir in the repo |
| Run one task, check in | Tell Claude "do task 3"; it uses **TodoWrite** to track and stops |
| Run all Tasks (waves) | An `/impl` command that loops the task list; subagents for parallel/independent tasks |
| Steering files (Kiro's persistent project rules) | **CLAUDE.md** + `@import`ed files |
| Sync (regenerate downstream on upstream edit) | A `/spec-sync` command you invoke after editing requirements |

**Key insight:** Kiro's "spec mode" is essentially a fixed set of prompts + a file
convention. Claude Code exposes exactly those as slash commands and CLAUDE.md, so
this ports cleanly — arguably *better*, because you control the prompts and check
them into the repo.

---

## 3. Two paths

### Path A — Drop-in community tool: `gotalab/cc-sdd` (recommended starting point)

Most battle-tested Kiro port (formerly `gotalab/claude-code-spec`, thousands of
stars). One command:

```bash
npx cc-sdd@latest
```

Installs ~17 slash commands implementing the exact Kiro pipeline:
`/kiro-spec-init` → `/kiro-spec-requirements` (EARS) → `/kiro-spec-design`
(Mermaid + File Structure Plan) → `/kiro-spec-tasks` (with literal `_Boundary:_` /
`_Depends:_` annotations per task) → `/kiro-impl` (long-running, one-task-at-a-time
implementation). Generates the same three files and is explicitly designed to be
**Kiro-spec-compatible**, so specs are portable between Kiro and Claude Code.
*(github.com/gotalab/cc-sdd, verified against README v3.0)*

`angelsen/claude-kiro` is a second, lighter port: stores
`requirements.md`/`design.md`/`tasks.md` per feature under
`.claude/specs/[feature]/` (or `.kiro/specs/[feature]/`), 3-phase checkpoint
workflow, enforces task-by-task execution via the native **TodoWrite** tool. Good
to read for how it's wired even if you don't adopt it.
*(github.com/angelsen/claude-kiro)*

**Why start here:** full opinionated flow immediately, and you can read the command
markdown to learn the patterns before customizing. Fastest way to give a team new
to Claude Code the Kiro feel on day one.

### Path B — Roll your own (~4 small files, fully owned, checked into the repo)

Best long-term for a team: the prompts become *your* team's standards and live in
the repo. Complete minimal setup below.

> **These two paths are independent — not layered.** The `starter-kit/` in this
> repo is Path B: hand-written `/spec-*` commands modeling the Kiro methodology
> directly. It is **not** based on cc-sdd, does not install or call it, and shares
> no code with it. cc-sdd (Path A) is a separate off-the-shelf tool with its own
> `/kiro-*` commands. Pick one. Don't assume the kit depends on cc-sdd.

**`.claude/commands/spec-requirements.md`**
```markdown
---
description: Phase 1 — generate EARS-format requirements for a feature
argument-hint: <feature-name> <short description>
---
Create `docs/specs/$1/requirements.md` for the feature described as: $ARGUMENTS

Write user stories with acceptance criteria in EARS notation:
`WHEN [condition/event] THE SYSTEM SHALL [expected behavior]`.

Cover happy path, edge cases, and error handling. Number every requirement
(R1, R2, …) so design and tasks can reference them.

When done, STOP and show me the file. Do not proceed to design until I approve.
```

**`.claude/commands/spec-design.md`**
```markdown
---
description: Phase 2 — technical design from approved requirements
argument-hint: <feature-name>
---
Read `docs/specs/$1/requirements.md`. Create `docs/specs/$1/design.md` with:
- Architecture overview + a Mermaid diagram
- Components and their interactions, data models, interfaces
- A File Structure Plan (which files change/are created)
- Each design decision tagged with the requirement IDs it satisfies

STOP and show me. Do not generate tasks until I approve.
```

**`.claude/commands/spec-tasks.md`**
```markdown
---
description: Phase 3 — break design into discrete, trackable tasks
argument-hint: <feature-name>
---
Read requirements.md and design.md under `docs/specs/$1/`. Create
`docs/specs/$1/tasks.md` as a numbered checklist. For each task include:
- A clear description and expected outcome
- `_Requirements:_` the requirement IDs it implements
- `_Depends:_` task numbers it depends on (blank if independent)

Use `- [ ]` checkboxes. Keep tasks small enough to review in one sitting.
STOP and show me before any implementation.
```

**`.claude/commands/spec-impl.md`** (the execution loop — pick one-at-a-time vs all)
```markdown
---
description: Implement spec tasks. Pass a task number for one-at-a-time, or "all".
argument-hint: <feature-name> <task-number | all>
---
Read `docs/specs/$1/tasks.md`.

If the second arg is a number: implement ONLY that task. Mark it `[x]` when done,
run the relevant tests, then STOP and report. (Kiro's default check-in mode.)

If the second arg is "all": load all tasks into TodoWrite, then implement them
respecting `_Depends:_` order — independent tasks can be delegated to parallel
subagents, dependent tasks wait. Mark each `[x]` as completed and run tests
after each. Report a summary at the end.
```

**`CLAUDE.md`** (the persistent "steering" layer) — add:
```markdown
## Spec-driven development
Features are specified before implementation in `docs/specs/<feature>/` as
requirements.md → design.md → tasks.md. Build them with /spec-requirements,
/spec-design, /spec-tasks (approve each phase before the next). Implement with
/spec-impl <feature> <task#> for one task, or /spec-impl <feature> all.
Never implement a feature that doesn't have an approved tasks.md.
```

That's the entire Kiro methodology, reproduced and owned. Commit
`.claude/commands/` and `docs/specs/` and the whole team is in sync — both the
*method* (commands) and the *specs* travel in git.

---

## 4. The execution model specifically

- **One at a time** = `/spec-impl feature 3`, or just "implement task 3 from the
  tasks file." Uses TodoWrite, does the one task, stops. Default-safe mode,
  mirrors Kiro's original intentional design.
- **All at once** = `/spec-impl feature all`. The `_Depends:_` annotations let
  Claude do Kiro's "wave" trick: independent tasks fan out to **subagents** (run
  in parallel), dependent ones serialize. Not Kiro's exact isolated-context wave
  scheduler, but the dependency-ordered behavior reproduces well.
- **Plan mode** (`shift+tab` to cycle in) complements the gates: enter plan mode
  during requirements/design so Claude proposes without touching files, then
  approve.

---

## 5. Caveats / things to know

- **Hooks** (`.claude/settings.json`) can enforce the discipline mechanically —
  e.g. a `PreToolUse` hook that blocks source edits when no `tasks.md` exists, or
  a `Stop` hook reminding you to update task checkboxes. The one piece Kiro
  doesn't really expose and Claude Code does.
- **No special git for specs** in either tool — plain markdown + commit.
- **EARS is just a convention** the prompt enforces; nothing technical depends on
  it. Keep it for testability or drop it if it feels heavy.
- **cc-sdd compatibility cuts both ways**: because it keeps Kiro-compatible spec
  files, you could pilot Claude Code on the *same* spec files your team already
  produces in Kiro at work — useful for a side-by-side evaluation.

---

## 6. The idiomatic Claude Code way (2026 research update)

> Added after a second research pass focused on how teams actually do
> spec/planning work in Claude Code — verified mostly against **primary Anthropic
> sources** (official docs + the "How Anthropic teams use Claude Code" white
> paper). This re-frames the Kiro port above as *one tier* of a broader spectrum.

**There is no official "spec-driven" framework in Claude Code — by design.** It
ships primitives (CLAUDE.md, slash commands, plan mode, subagents, hooks, Tasks)
and expects you to compose the workflow. So a Kiro-style kit isn't un-idiomatic;
it's just the heavyweight end of a spectrum Anthropic itself documents.

### Anthropic's own recommended spectrum

- **Skip planning** for small clear-scope work — *"if you could describe the diff
  in one sentence, skip the plan."* (official best-practices, verbatim)
- **Plan mode** for uncertain approaches, multi-file changes, or unfamiliar code.
- **Write a spec** for larger features: Claude *interviews you* (AskUserQuestion),
  writes a complete self-contained **`SPEC.md`**, then you **start a fresh session
  to execute it** against that file. Note: Anthropic's heavyweight tier is a
  *single* SPEC.md, not Kiro's three files.
- **CLAUDE.md as a living spec** — teams have Claude summarize each session and
  continuously refine CLAUDE.md (white paper). Anthropic's own teams "write,
  review, and execute specifications in markdown stored in the codebase."

### The 4-level ladder (what this kit is built around)

| Tier | When | Tool |
|---|---|---|
| 1. Just do it | one-sentence diff | ask directly |
| 2. Plan mode | uncertain/multi-file, one sitting | `shift+tab` |
| 3. tasks.md only | durable, committed, multi-step checklist | `/spec-tasks` standalone → `/spec-impl` |
| 4. Full spec | big/shared/multi-session feature | full `/spec-*` flow |

A spec earns its cost when multiple sessions/developers/agents touch the same
feature, outputs must match a shared design, or drift cost > authoring cost.

### Native Tasks changed the persistence story (Jan 2026)

Claude Code replaced session-volatile Todos with a **Tasks** primitive
(`TaskCreate`/`TaskUpdate`/`TaskGet`/`TaskList`, default since v2.1.142). Tasks
**persist across sessions and subagents** on the local filesystem
(`~/.claude/tasks`) — but are **machine-local and never uploaded**, so they don't
travel in git. That means:

- The old reason practitioners hand-rolled `TODO.md`/`tasks.md` files (the built-in
  list didn't survive `/clear` or new sessions) is now *partly* handled natively.
- But a **committed `tasks.md` still has a distinct job**: it's the team-shared,
  reviewable, git-tracked plan. Native Tasks = your local live state; `tasks.md` =
  the artifact the team reviews. They coexist.

### Community signal

Consistent critique that heavyweight Kiro-style frameworks (e.g. "spec kitty")
over-plan small tasks; multiple guides say skip spec-driven dev entirely for
single-file bug fixes ("writing a specification costs more than it saves").
Practitioner persistence patterns in the wild: `TODO.md` (global + per-project),
a minimalist tasks.md/requirements.md/session.md trio, saving plan-mode output to
a repo `_plans/` folder, and the `planning-with-files` skill. Many predate native
Tasks and were workarounds for session-volatility.

### Two things the research *refuted* (don't repeat these)

- Claude Code does **not** auto-enter plan mode from prompt intent — you toggle it.
- "CLAUDE.md + auto-memory as two officially-documented complementary memory
  systems" was not supported as stated.

**Net:** keep this Kiro-style kit, but treat it as tier 3–4 only. For tiers 1–2,
lean on vibe-coding and plan mode. The kit's real edge over native primitives is
**git-committed, team-reviewable specs** — which native Tasks deliberately is not.

---

## Recommendation

Optionally run `npx cc-sdd@latest` in a *throwaway* repo just to feel the full
flow and borrow ideas — but the kit in `starter-kit/` is the recommended path:
hand-written `/spec-*` commands that are yours, checked in, and dependency-free
(no relation to cc-sdd). Then right-size: use the full flow only for tier 3–4
work, and lean on plan mode + vibe-coding below that.

---

## Sources

Primary (verified):
- kiro.dev/docs/specs/
- kiro.dev/docs/specs/feature-specs/
- kiro.dev/docs/specs/feature-specs/requirements-first/
- kiro.dev/docs/specs/feature-specs/tech-design-first/
- kiro.dev/docs/specs/best-practices/
- kiro.dev/docs/getting-started/first-project/
- kiro.dev/docs/editor/source-control/
- kiro.dev/blog/run-all-tasks/
- kiro.dev/blog/faster-smarter-specs/
- kiro.dev/changelog/ide/0-12/

Community ports (verified):
- github.com/gotalab/cc-sdd
- github.com/angelsen/claude-kiro
- angelsen.github.io/claude-kiro/getting-started.html

Secondary / commentary:
- augmentcode.com/guides/claude-code-spec-driven-development
- builder.aws.com — experience with Kiro's SDD methodology
- martinfowler.com (referenced for three-file structure)

Claude Code conventions, 2026 update (primary):
- code.claude.com/docs/en/best-practices (explore-plan-code-commit, SPEC.md, one-sentence-diff rule)
- code.claude.com/docs/en/agent-sdk/todo-tracking (Task tools default in v2.1.142)
- code.claude.com/docs/en/agent-teams (Tasks persist locally, never uploaded)
- "How Anthropic teams use Claude Code" white paper (CLAUDE.md as living spec, markdown specs in codebase)
- Boris Cherny announcement — Tasks primitive, Jan 22 2026

Claude Code conventions, 2026 update (community/blogs):
- Nick Tune — minimalist tasks.md/requirements.md/session.md workflow
- samfrenchblog.com — TODO.md across sessions
- github.com/othmanadi/planning-with-files
- numustafa (Medium) — plan mode + repo _plans/ folder
