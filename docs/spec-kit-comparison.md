# GitHub spec-kit vs. this starter kit

> Findings from a look at [github/spec-kit](https://github.com/github/spec-kit)
> (researched 2026-07-01/02), written up as a comparison against this repo's
> Kiro-style starter kit. TL;DR: same spec-driven-development family, different
> lineage and trade-offs — spec-kit is an agent-agnostic external toolkit;
> this kit is Claude Code–native with zero tooling. A few spec-kit ideas are
> worth mining (noted at the end), but they solve different sharing problems.

## What spec-kit is (and isn't)

**It is not a Claude/Anthropic project.** Spec-kit is **GitHub's** open-source
toolkit for Spec-Driven Development (SDD). It's agent-agnostic: Claude Code is
one of 30+ supported integrations, alongside GitHub Copilot, Gemini CLI, Cursor,
Codex CLI, Windsurf, Qwen Code — and even Kiro CLI itself.

Its stated philosophy is stronger than Kiro's: *"specifications become
executable, directly generating working implementations rather than just
guiding them."* Code serves the spec, not the other way around.

### Architecture: two pieces

1. **The Specify CLI** — a Python 3.11+ tool (`uv tool install specify-cli
   --from git+https://github.com/github/spec-kit.git`). `specify init`
   bootstraps a project: it lays down templates, helper scripts
   (bash/PowerShell), and the slash-command files for whichever agent you pick
   (`--integration claude`, etc.).
2. **Templates + scripts** — the actual workflow content the commands consume,
   installed under `.specify/`.

### The workflow commands

Core flow, in order:

| Command | Does | Rough analog here |
|---|---|---|
| `/speckit.constitution` | Project-wide governing principles, persisted in `.specify/memory/constitution.md`; every later phase is checked against it | `CLAUDE.md` + `spec-workflow.md` rules (implicit, not a gated artifact) |
| `/speckit.specify` | Functional requirements + user stories — the *what/why*, deliberately tech-free | `/spec-requirements` (but free-form user-story style, **not EARS**) |
| `/speckit.clarify` | Structured questioning pass to de-ambiguate the spec before planning | Baked into each phase's "clarify before drafting" rule |
| `/speckit.plan` | Technical implementation strategy incl. tech stack | `/spec-design` |
| `/speckit.tasks` | Ordered, actionable task breakdown | `/spec-tasks` |
| `/speckit.implement` | Executes all tasks | `/spec-impl <feature> all` |

Optional/supporting commands:

- `/speckit.analyze` — cross-artifact consistency + coverage check
  (spec ↔ plan ↔ tasks). Closest thing to `/spec-sync`, though it *reports*
  drift rather than reconciling it.
- `/speckit.converge` — assesses the *codebase* against spec/plan/tasks and
  identifies remaining work (brownfield catch-up).
- `/speckit.checklist` — generates custom quality checklists validating the
  requirements.
- `/speckit.taskstoissues` — converts the task list into GitHub issues.

### Artifacts and repo mechanics

```
.specify/
  memory/constitution.md
  scripts/bash/            # create-new-feature.sh, setup-plan.sh, ...
  templates/               # spec-template.md, plan-template.md, tasks-template.md
specs/
  001-feature-name/
    spec.md  plan.md  tasks.md
    research.md  data-model.md  contracts/
```

- **Branch-per-feature is built in**: `/speckit.specify` runs a helper script
  that creates a numbered feature branch (e.g. `001-create-taskify`) and the
  matching `specs/001-.../` directory. This kit deliberately leaves git
  workflow to the user (as does Kiro — see `research-findings.md`: "No special
  version control for specs").
- **Richer per-feature artifacts**: beyond the spec/plan/tasks trio, the plan
  phase can emit `research.md`, `data-model.md`, and API `contracts/`.
- **Customization layer**: *extensions* (new commands, e.g. Jira integration),
  *presets* (reskin existing workflows to org standards), and *bundles*
  (role-based sets), resolved with priority ordering
  (project overrides > presets > extensions > core). This is spec-kit's answer
  to the sharing problem the starter-kit README solves with user-level
  commands / `@import` / plugins.

## Head-to-head

| Dimension | spec-kit (GitHub) | This kit |
|---|---|---|
| Origin / lineage | GitHub's own SDD take | Kiro replication |
| Agent support | 30+ agents, lowest-common-denominator templates | Claude Code only, leans on native primitives |
| Install | Python CLI (`uv`/`pipx`), Python 3.11+, helper scripts | Copy `.claude/` + one `CLAUDE.md` import line; plain markdown + git |
| Requirements format | Free-form user stories / acceptance criteria | **EARS** notation (Kiro's rigor) |
| Phase gating | Sequential commands; approval discipline is on the user | Explicit Kiro gating model: each command stops for approval; optional hooks enforce |
| Task execution | `/speckit.implement` runs everything | `/spec-impl <n>` check-in mode **or** `all` (dependency-ordered) — mirrors Kiro's two modes |
| Drift handling | `/speckit.analyze` (report), `/speckit.converge` (code vs. spec) | `/spec-sync` (actively reconciles downstream artifacts) |
| Persistent principles | Explicit `constitution.md` artifact, versioned, checked per phase | `CLAUDE.md` / `spec-workflow.md` guidance (always loaded, but not an artifact phases validate against) |
| Git workflow | Branch-per-feature scripted in | Left to the user, intentionally |
| Right-sizing | Not addressed — every feature gets the full flow | Explicit 4-tier "when to use what" ladder; skip specs for small work |
| Team sharing | Extensions/presets/bundles via the CLI registry | User-level commands, `@import`, or Claude Code plugins |
| GitHub integration | `/speckit.taskstoissues` | None (tasks.md is the committed artifact) |

## Assessment

- **Keep this kit if you're all-in on Claude Code.** Spec-kit's
  multi-agent portability is its headline feature and its main cost: templates
  can't assume plan mode, hooks, `@import`, native Tasks, or any other Claude
  Code specific. This kit exploits all of them, needs no Python toolchain, and
  the *method* travels in the repo via git alone — the same reasoning as the
  "why roll our own vs. adopt cc-sdd" section in the guide.
- **Spec-kit is the better fit** for mixed-agent teams (some on Copilot, some
  on Claude, some on Gemini) that want one spec process, or orgs that want the
  extensions/presets governance layer.

### Ideas worth stealing

1. **The constitution as a checked artifact.** Making project principles an
   explicit file that each phase validates against (rather than ambient
   CLAUDE.md guidance) is spec-kit's most-cited contribution. A cheap version:
   have `/spec-design` and `/spec-tasks` end with a "conflicts with
   spec-workflow rules?" self-check.
2. **`/speckit.converge`.** Assessing an existing codebase against a spec to
   find remaining/undone work is a brownfield move neither Kiro nor this kit
   has; `/spec-sync` reconciles artifacts with each other, not with the code.
3. **`taskstoissues`.** If a team already lives in GitHub issues, a
   `/spec-issues` command mapping `tasks.md` checkboxes to issues would be a
   small, useful addition.

## Sources

- [github/spec-kit](https://github.com/github/spec-kit) (README, fetched 2026-07-02)
- [Spec Kit documentation](https://github.github.com/spec-kit/)
- [GitHub Blog — Spec-driven development with AI: get started with a new open source toolkit](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)
- [spec-kit/spec-driven.md — the SDD philosophy doc](https://github.com/github/spec-kit/blob/main/spec-driven.md)
- [Microsoft for Developers — Diving into spec-driven development with GitHub Spec Kit](https://developer.microsoft.com/blog/spec-driven-development-spec-kit)
- [LogRocket — Exploring spec-driven development with the new GitHub Spec Kit](https://blog.logrocket.com/github-spec-kit/)
- [Tessl — A look at Spec Kit](https://tessl.io/blog/a-look-at-spec-kit-githubs-spec-driven-software-development-toolkit/)
