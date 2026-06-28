# Research findings (preserved)

> Durable record of two `deep-research` workflow runs (2026-06-27) so the verified
> findings can be reused/extended **without re-running the expensive workflows**
> (~90+ Opus subagents and ~1M+ tokens each — together the bulk of a ~$50 spend).
> The prose conclusions live in `kiro-spec-driven-dev-in-claude-code.md`; this file
> keeps the underlying verified claims, confidence, sources, and caveats.

---

## Run 1 — Kiro mechanics + replicating them in Claude Code

**Stats:** 20 sources fetched, 81 claims extracted, 25 verified, 24 confirmed,
1 killed. (Note: Run 1's final synthesis step glitched; these were recovered from
the adversarial-verify transcripts.)

### Confirmed findings

- **Three sequential artifacts.** `requirements.md` (user stories + acceptance
  criteria in **EARS**: `WHEN [condition] THE SYSTEM SHALL [behavior]`),
  `design.md` (technical architecture, sequence diagrams, components/interactions),
  `tasks.md` (discrete, trackable tasks with dependencies).
  *(kiro.dev/docs/specs, /feature-specs)*
- **Gated phases (standard mode).** "The standard spec flow has three phases:
  requirements, design, tasks. You approve each one before the next begins."
  *(kiro.dev/blog/faster-smarter-specs)*
- **Quick Plan mode** generates all three without approval gates; **Design-First**
  variant (Design → Requirements → Tasks) exists alongside default Requirements-First.
  *(kiro.dev/docs/specs/feature-specs/tech-design-first)*
- **Execution: individual or all-at-once.** "Execute tasks individually or run all
  tasks." Run all Tasks builds a dependency graph and groups independent tasks into
  **waves** that run concurrently. *(kiro.dev/docs/specs/best-practices, /blog/run-all-tasks)*
- **One-at-a-time was intentional.** "You had to check in with the agent after
  every task. That wasn't an oversight. It was intentional" (visibility/control).
  *(kiro.dev/blog/run-all-tasks)*
- **No special version control for specs.** Source-control docs cover only ordinary
  git; specs are plain markdown. "Sync" regenerates downstream artifacts, not a git
  op. *(kiro.dev/docs/editor/source-control — verified absence)*
- **cc-sdd** (community port): `requirements.md` EARS + acceptance criteria;
  `design.md` architecture w/ Mermaid + File Structure Plan; `tasks.md` with literal
  `_Boundary:_`/`_Depends:_` markers. *(github.com/gotalab/cc-sdd)*
- **claude-kiro** (community port): three files under `.kiro/specs/[feature]/`,
  task-by-task execution via native TodoWrite. *(github.com/angelsen/claude-kiro)*

### Killed claim

- ✗ "Kiro tasks are executed one-at-a-time by clicking individual task items, NOT
  all at once" (0-3) — refuted; Run all Tasks now exists.

### Sources (Run 1)

Primary (kiro.dev): /docs/specs/, /docs/specs/feature-specs/,
/feature-specs/requirements-first/, /feature-specs/tech-design-first/,
/docs/specs/best-practices/, /docs/getting-started/first-project/,
/docs/editor/source-control/, /blog/run-all-tasks/, /blog/faster-smarter-specs/,
/changelog/ide/0-12/.
Community ports: github.com/gotalab/cc-sdd, github.com/angelsen/claude-kiro,
angelsen.github.io/claude-kiro/getting-started.html.
Commentary/contrarian: augmentcode.com guide, builder.aws.com, Martin Fowler,
medium (francoisdexemple Kiro deep-dive), infoworld (Kiro pricing-glitch),
yehudacohen substack, buildthisnow, dev.to (aws-builders).

---

## Run 2 — How teams actually do spec/planning work in Claude Code (2026)

**Stats:** 9 sources fetched, 41 claims, 25 verified, 23 confirmed, 2 killed,
18 after synthesis. (Synthesis came through intact this run.)

### Confirmed findings (with confidence + vote)

- **[high, 3-0] Explore → Plan → Code → Commit** is Anthropic's official four-phase
  workflow (plan mode for read-only explore/plan, then implement, then commit/PR).
  *(code.claude.com/docs/en/best-practices)*
- **[high, 3-0] Tiered approach is official.** Skip plan mode for small clear-scope
  tasks; plan when uncertain/multi-file/unfamiliar. Verbatim: *"if you could
  describe the diff in one sentence, skip the plan."* *(best-practices)*
- **[high, 3-0] Big-feature pattern = single SPEC.md.** Claude interviews you via
  AskUserQuestion, writes a complete self-contained `SPEC.md`, then you **start a
  fresh session to execute it**. Specs name files/interfaces, state out-of-scope,
  end with end-to-end verification. *(best-practices)*
- **[high, 3-0] CLAUDE.md as a living spec.** Teams have Claude summarize sessions
  and suggest improvements — a continuous loop refining CLAUDE.md. *(Anthropic white
  paper "How Anthropic teams use Claude Code")*
- **[high, 2-1] Anthropic teams write/review/execute markdown specs in the codebase**
  (e.g. "dependant" app) → contributions in days not weeks. *(white paper)*
- **[high, 3-0] Autonomy sized to task type:** auto-accept for peripheral/prototype
  code; synchronous supervision for core logic / critical fixes. *(white paper)*
- **[high, 3-0] Native Tasks primitive (Jan 22, 2026).** Upgraded from Todos;
  spans beyond one session, coordinates across sessions/subagents; **persists on the
  filesystem (`~/.claude/tasks`), never uploaded**, so resumed sessions keep tasks.
  *(Boris Cherny; code.claude.com/docs/en/agent-teams)*
- **[high, 3-0] Task tools default since v2.1.142** (`TaskCreate`/`TaskUpdate`/
  `TaskGet`/`TaskList`) replacing single `TodoWrite`; re-enable old via
  `CLAUDE_CODE_ENABLE_TASKS=0`. *(code.claude.com/docs/en/agent-sdk/todo-tracking)*
- **[high, 3-0] Auto-todo triggers:** 3+ distinct actions, user multi-item lists,
  non-trivial progress-tracking work, explicit user request. *(todo-tracking)*
- **[medium] Persistent on-disk plan files** (plan.md/todo.md) arose because the old
  built-in todo list didn't survive `/clear` or new sessions. *(Nick Tune)* — now
  partly superseded by native Tasks.
- **[medium] Minimalist three-file workflow:** tasks.md (todo + status) /
  requirements.md (detail) / session.md (progress for resume). *(Nick Tune)*
- **[medium] planning-with-files skill:** task_plan.md / findings.md / progress.md
  as "filesystem = disk" memory; hooks re-read on new session. *(github.com/othmanadi/planning-with-files)*
- **[medium] Two-level TODO.md** (global + per-project) with status markers, CLAUDE.md
  instructing "always check TODO.md first." *(samfrenchblog.com)*
- **[medium, 3-0] Three-layer context model:** TODO.md (live state) vs CLAUDE.md
  (conventions) vs MCP Memory (long-term) — "sprint board vs handbook vs wiki."
- **[medium, 3-0] Save plan-mode output to repo `_plans/` folder** as resumable docs
  (plansDirectory setting, skills, GH feature requests). *(numustafa, Medium)*
- **[medium, 3-0] Critique of heavyweight spec frameworks.** "spec kitty" over-plans
  small tasks; skip SDD for single-file bug fixes — "a spec costs more than it saves."
  *(Nick Tune; augmentcode.com guide)*
- **[medium, 3-0] Spec earns its cost when** multiple sessions/devs/agents touch the
  feature, outputs must match a shared design, or drift cost > authoring cost.
  *(augmentcode.com guide; corroborated by Microsoft Dev Blog, Martin Fowler)*

### Killed claims

- ✗ [0-3] "CLAUDE.md + auto-memory are two officially-documented complementary
  memory systems." Not supported as stated.
- ✗ [0-3] "Claude Code auto-enters plan mode from prompt intent." It doesn't — you
  toggle it (shift+tab).

### Caveats

- **Time-sensitive.** Native Tasks (Jan 22, 2026) + the SDK default switch from
  TodoWrite to Task tools (v2.1.142) partially supersede the manual TODO.md/tasks.md
  conventions; old and new coexist. Several blog conventions predate Tasks.
- **Source quality mixed.** Anthropic-official findings are high-confidence (primary
  docs, white paper, Boris Cherny). Practitioner conventions rest on individual blogs
  / one GitHub repo with vendor-guide corroboration — emerging, not canonical.
- Status markers and filenames are **not standardized** (each tool/blog defines its
  own). planning-with-files session-recovery is that skill's hooks, not native CC.

### Open questions

- Are practitioners migrating from manual TODO.md to native Tasks, or going hybrid
  (Tasks for live state + markdown specs for design)?
- Will a naming standard emerge (SPEC.md vs tasks.md vs plan.md) or stay fragmented?
- How do native Tasks + repo `_plans/` interact in multi-dev shared repos (merge
  conflicts, ownership, review)?

### Sources (Run 2)

Primary: code.claude.com/docs/en/best-practices,
code.claude.com/docs/en/agent-sdk/todo-tracking, code.claude.com/docs/en/agent-teams,
Anthropic white paper (www-cdn.anthropic.com/58284b19e702b49db9302d5b6f135ad8871e7658.pdf),
Boris Cherny (threads.com/@boris_cherny/post/DT15_lHjmWS).
Secondary/blog: augmentcode.com guide, github.com/othmanadi/planning-with-files,
samfrenchblog.com (TODO.md), Nick Tune (medium, minimalist workflow),
numustafa (medium, plan mode + _plans/).

---

## How to reuse this

To extend or revise anything in `claude-code-spec` later, start from this file
instead of re-running deep-research. Only re-run a workflow if you specifically need
**fresher** data than 2026-06-27 (the space is moving fast — native Tasks is recent).
If you do, scope it narrowly to the new question to keep cost down, and consider
running it on Sonnet rather than Opus.
