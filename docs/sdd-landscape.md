# SDD tool landscape

> One-page survey of the spec-driven-development (SDD) tools encountered while
> building this kit (researched 2026-07-06, star counts and activity as of that
> date). Each entry: what it is, its SDD *style*, core features, and where it
> lives. Deeper material elsewhere in this repo: `spec-kit-comparison.md`
> (spec-kit head-to-head), `research-findings.md` (Kiro mechanics, verified),
> and the command-name collision spike in `TODOs-completed.md`.

## At a glance

| Tool | Stars | Status | SDD style (short) | Agents | Install |
|---|---|---|---|---|---|
| [Kiro](https://kiro.dev) | — | active (AWS) | gated 3-artifact phases (the original) | Kiro IDE/CLI | IDE |
| [spec-kit](https://github.com/github/spec-kit) | ~118k | active (GitHub) | constitution + executable specs | 30+ agents | Python CLI |
| [Get Shit Done](https://github.com/gsd-build/get-shit-done) → [gsd-core](https://github.com/open-gsd/gsd-core) | ~65k → ~6k | archived → active successor | specs as context engineering | CC-only → multi-runtime | npx |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | ~59k | active | change proposals / delta specs | 25+ agents | npm |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | ~50k | active | agile agent personas + story files | multi (incl. web UIs) | npx |
| [Agent OS](https://github.com/buildermethods/agent-os) | ~5k | maintained | standards-first spec shaping | multi | script/copy |
| [spec-workflow-mcp](https://github.com/Pimzino/spec-workflow-mcp) | ~4.3k | active (hiatus announced) | Kiro-style gates + approval dashboard | any MCP client | npx (MCP) |
| [claude-code-spec-workflow](https://github.com/Pimzino/claude-code-spec-workflow) | ~3.8k | dormant (superseded) | Kiro-style slash commands | Claude Code | npm |
| [cc-sdd](https://github.com/gotalab/cc-sdd) | ~3.5k | active | Kiro port with validation gates | CC + others | npm |
| [spec-based-claude-code](https://github.com/papaoloba/spec-based-claude-code) | 133 | dormant | Kiro-style, marker-file gates | Claude Code | copy files |
| [claude-kiro](https://github.com/angelsen/claude-kiro) | 6 | inactive | minimal Kiro port | Claude Code | copy files |
| [claude-spec](https://github.com/zircote/claude-spec) | 1 | archived | heavy governance suite | Claude Code | plugin |

Three broad families show up:

1. **Kiro lineage** — gated requirements → design → tasks artifacts with human
   approval between phases (Kiro, both Pimzino tools, cc-sdd, papaoloba,
   claude-kiro, **this kit**).
2. **Spec-as-source-of-truth** — the spec outranks the code; tooling checks
   code against it (spec-kit, OpenSpec).
3. **Process frameworks** — SDD embedded in a larger method: agent teams
   (BMAD), context engineering (GSD), house standards (Agent OS).

---

## Kiro ([kiro.dev](https://kiro.dev))

AWS's agentic IDE and the origin of the workflow this kit replicates. Its
"specs" feature turns a feature request into three sequential artifacts —
`requirements.md` (EARS notation), `design.md`, `tasks.md` — with explicit
human approval gating each phase.

**SDD style:** the canonical gated three-artifact flow; requirements-first by
default, with Design-First and ungated Quick Plan variants.

**Core features:**
- EARS-format requirements (`WHEN [condition] THE SYSTEM SHALL [behavior]`)
- Approval gate after each artifact (standard mode)
- Task execution one-at-a-time or "Run all Tasks" (dependency graph, concurrent waves)
- "Sync" regenerates downstream artifacts after upstream edits
- Specs are plain markdown in the repo; no special version control

## GitHub spec-kit ([github/spec-kit](https://github.com/github/spec-kit), ~118k ★)

GitHub's own SDD toolkit and the biggest name in the space. Agent-agnostic: a
Python CLI (`specify init`) installs templates, helper scripts, and
slash-command files for whichever of 30+ agents you use. Full comparison in
`spec-kit-comparison.md`.

**SDD style:** "specifications become executable" — the spec outranks the
code. A versioned `constitution.md` of project principles is checked at every
phase; sequential `/speckit.*` commands, but approval discipline is on the
user rather than mechanically gated.

**Core features:**
- Constitution as a first-class, phase-validated artifact
- `/speckit.specify` → `clarify` → `plan` → `tasks` → `implement` core flow
- Richer artifacts: `research.md`, `data-model.md`, API `contracts/` per feature
- `/speckit.analyze` (cross-artifact drift report) and `/speckit.converge` (code-vs-spec brownfield catch-up)
- `/speckit.taskstoissues` — task list → GitHub issues
- Extensions/presets/bundles customization layer; branch-per-feature scripted in

## Get Shit Done ([gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done), ~65k ★, archived) → GSD Core ([open-gsd/gsd-core](https://github.com/open-gsd/gsd-core), ~6k ★)

"Meta-prompting, context engineering and spec-driven development" — broader
than pure SDD. Its core enemy is *context rot*: specs and plans exist so every
agent invocation runs in fresh, small context. The original (largest star
count in this list) was archived June 2026; development continues as GSD Core,
which went multi-runtime.

**SDD style:** project → requirements → roadmap → phased execution. Per
milestone: Discuss → Plan → Execute → Verify → Ship, with human checkpoints at
the discuss/plan boundaries rather than formal sign-off artifacts.

**Core features:**
- `/gsd-*` commands (new-project, discuss/plan/execute phase, …)
- Fresh-context subagent "waves" per task batch — the context-engineering centerpiece
- Durable state in structured files (`STATE.md`, `CONTEXT.md`, planning docs)
- Atomic per-task commits; verify-before-ship step
- GSD Core runs on Claude Code, OpenCode, Codex, Copilot, Cursor, Windsurf, and more

## OpenSpec ([Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec), ~59k ★)

A lightweight planning layer between requirements and code, agent-agnostic
(25+ assistants). Distinctive for rejecting rigid phase gates — "fluid not
rigid, iterative not waterfall."

**SDD style:** change proposals / delta specs. Each change is a bundle under
`openspec/` — `proposal.md`, spec *deltas*, `design.md`, `tasks.md` — and
archiving a completed change merges its deltas into the living canonical spec
set. No formal approval gate; the human drives the propose → apply → archive
loop.

**Core features:**
- `/opsx:explore` — think through a problem with the AI before proposing
- `/opsx:propose` → `apply` → `archive` change loop
- Living specs: the canonical spec base evolves by absorbing archived deltas
- npm CLI generates per-agent slash commands; dashboard for visualizing changes
- "Stores" (beta) — cross-repo planning for team coordination

## BMAD-METHOD ([bmad-code-org/BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD), ~50k ★)

"Breakthrough Method for Agile AI-Driven Development" — a full agile-team
simulation with 12+ AI agent personas (Analyst, PM, Architect, Developer, UX,
…) carrying a project from brainstorming to implementation. The
heaviest-process option in this list; also broader than software (game-dev and
creative expansion packs). License note: README says MIT but GitHub reports
custom terms around the trademark.

**SDD style:** agent-persona + PRD-driven, two macro phases. Planning personas
produce rich artifacts (briefs, PRDs, UX specs, architecture docs); dev
personas shred them into hyper-detailed **story files** carrying full context
into implementation. The human steers as product owner between handoffs rather
than through one hard gate.

**Core features:**
- 12+ named personas; "Party Mode" multi-persona sessions
- 34+ built-in workflows, brainstorming → implementation
- Story files embed context + acceptance criteria so dev agents skip re-discovery
- Web bundles: do planning in Gemini/ChatGPT web UIs, then switch to an IDE agent
- v6 "scale-adaptive" planning sizes the process to the project

## Agent OS ([buildermethods/agent-os](https://github.com/buildermethods/agent-os), ~5k ★)

Brian Casel's system for making agents code the way *your team* codes. The
main pain it targets is agents ignoring house conventions, more than spec
ceremony itself.

**SDD style:** standards-first, three context layers — **Standards** (coding
patterns, auto-extracted from your codebase in v3), **Product**
(mission/roadmap), **Specs** (per-feature spec + tasks). Specs are "shaped"
with relevant standards injected; review is informal, no mechanical gate.

**Core features:**
- Discover Standards: analyzes an existing codebase and extracts its patterns
- Contextual injection: only the standards relevant to the task at hand
- `/agent-os:shape-spec` guided spec creation; profiles per project type/stack
- Plain shell + markdown — no server; tool- and language-agnostic

## spec-workflow-mcp ([Pimzino/spec-workflow-mcp](https://github.com/Pimzino/spec-workflow-mcp), ~4.3k ★)

An MCP server that imposes a Kiro-style spec workflow on any MCP-capable
agent, paired with a web dashboard and VSCode extension where humans review
and approve. The active successor to claude-code-spec-workflow (below).
Maintainer announced a temporary hiatus in mid-2026; GPL-3.0 (the only
copyleft license here).

**SDD style:** gated Kiro-style Requirements → Design → Tasks in
`.spec-workflow/specs/`, with a **first-class approval gate**: specs are
submitted through the dashboard/extension with feedback and revision tracking
before implementation may proceed. Steering documents for persistent context.

**Core features:**
- MCP server — works with Claude Code, Cursor, Cline, Windsurf, Continue, Codex, …
- Live dashboard: spec/task progress, approvals, implementation logs
- VSCode sidebar extension as an alternative approval surface
- Formal revision-tracked approvals; archive tree; i18n (11 languages), Docker

## claude-code-spec-workflow ([Pimzino/claude-code-spec-workflow](https://github.com/Pimzino/claude-code-spec-workflow), ~3.8k ★, dormant)

Pimzino's original Claude Code-specific tool and the ancestor of
spec-workflow-mcp — README now redirects there. Matters to this kit mainly as
the one tool that ever shipped flat `spec-*` commands (see the collision spike
in `TODOs-completed.md`): current names are `/spec-create`, `/spec-execute`,
`/spec-status`, `/spec-list`; releases through ~July 2025 literally installed
`/spec-requirements`, `/spec-design`, `/spec-tasks`.

**SDD style:** gated Kiro-style phases via slash commands, plus a separate
bug-fix workflow (Report → Analyze → Fix → Verify).

**Core features:**
- Spec flow + bug flow commands; auto-generated per-task commands
- Steering docs (`product.md`, `tech.md`, `structure.md`)
- Real-time WebSocket dashboard

## cc-sdd ([gotalab/cc-sdd](https://github.com/gotalab/cc-sdd), ~3.5k ★)

The most faithful community port of Kiro's workflow to Claude Code (formerly
`gotalab/claude-code-spec`). Closest neighbor to this kit in both style and
command vocabulary — its `/kiro-spec-requirements|design|tasks|impl` share
four of this kit's five command suffixes behind the `kiro` namespace.

**SDD style:** gated Kiro-style three artifacts with EARS requirements, plus
explicit validation commands between phases.

**Core features:**
- `/kiro-spec-init` → `requirements` → `design` → `tasks` → `impl` flow (legacy `/kiro:spec-*` still installable)
- EARS acceptance criteria; `design.md` with Mermaid + File Structure Plan; `tasks.md` with `_Boundary:_`/`_Depends:_` markers
- Steering docs (`/kiro-steering`) and validation gates (`/kiro-validate-gap|design|impl`)
- Batch and quick modes (`/kiro-spec-batch`, `/kiro-spec-quick`)

## Smaller / inactive

- **[spec-based-claude-code](https://github.com/papaoloba/spec-based-claude-code)**
  (papaoloba, 133 ★, dormant since July 2025) — nine `/spec:*` commands
  implementing Kiro-style phases, with marker files (`.requirements-approved`)
  mechanically blocking phase progression, and multi-spec switching. Pure
  copied markdown; no license.
- **[claude-kiro](https://github.com/angelsen/claude-kiro)** (angelsen, 6 ★) —
  minimal Kiro port: `/spec:create|implement|review`, three files under
  `.kiro/specs/[feature]/`, task execution via native todo tracking.
- **[claude-spec](https://github.com/zircote/claude-spec)** (zircote, 1 ★,
  archived Feb 2026) — governance-heavy plugin: Socratic requirements
  elicitation, seven-document suite per project, hard approve/reject gate that
  can block writes without an approved spec. Never found an audience.

---

## Where this kit sits

Kiro lineage, Claude Code-native, zero tooling: five flat `/spec-*` commands
(`requirements`, `design`, `tasks`, `impl`, `sync`), EARS requirements,
explicit approval gates, specs as plain markdown in `specs/<feature>/`. The
deliberate trade against everything above: no CLI, no server, no dashboard —
the method travels in the repo via git alone, and leans on Claude Code
primitives (plan mode, hooks, `@import`) the agent-agnostic tools can't
assume. Closest relatives: cc-sdd (same lineage, more machinery) and spec-kit
(same ambition, different lineage — see `spec-kit-comparison.md`).
