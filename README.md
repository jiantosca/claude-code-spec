# claude-code-spec

Kiro-style **spec-driven development** for [Claude Code](https://claude.com/claude-code):
a workflow that turns a feature idea into `requirements.md` → `design.md` →
`tasks.md` and then implements it task-by-task — built entirely from Claude
Code's native primitives (slash commands, `CLAUDE.md`, hooks).

> **What this is (and isn't).** This is a personal project — built to explore
> how far those native primitives can go at replicating Kiro's spec workflow,
> with no CLI, server, or install step. If you want a maintained SDD framework
> with a community behind it, see
> [`docs/sdd-landscape.md`](docs/sdd-landscape.md) for a survey of the
> ecosystem (spec-kit, OpenSpec, cc-sdd, …).

## What's here

### → [`starter-kit/`](starter-kit/) — the kit you actually install
Drop-in `/spec-*` slash commands, the workflow rules Claude follows, optional
hooks, and a worked `example-feature/` spec. Start here if you just want to use
it. See the [starter-kit README](starter-kit/README.md) for install steps, and
[`MIGRATING-FROM-KIRO.md`](starter-kit/MIGRATING-FROM-KIRO.md) if you're coming
from Kiro.

### → [`docs/kiro-spec-driven-dev-in-claude-code.md`](docs/kiro-spec-driven-dev-in-claude-code.md) — the guide
The prose write-up: what Kiro actually does and how to replicate it in Claude
Code. Read this to understand *why* the kit is built the way it is.

### → [`docs/sdd-landscape.md`](docs/sdd-landscape.md) — the SDD tool landscape
One-page survey of every spec-driven-development tool encountered along the
way (spec-kit, OpenSpec, BMAD, GSD, cc-sdd, …): what each is, its SDD style,
core features, and where this kit sits among them.

### → [`docs/spec-kit-comparison.md`](docs/spec-kit-comparison.md) — how this differs from GitHub's spec-kit
Findings on [github/spec-kit](https://github.com/github/spec-kit) (GitHub's
agent-agnostic SDD toolkit) and a head-to-head against this kit: trade-offs,
when to prefer which, and spec-kit ideas worth borrowing.

### → [`docs/research-findings.md`](docs/research-findings.md) — the raw notes
The verified findings behind the guide (sources, confidence, caveats), preserved
so they can be reused without re-running the original research.

## Quick start

```
Copy starter-kit/.claude/ and starter-kit/CLAUDE.md into your repo,
then run /spec-requirements <feature> in Claude Code.
```

Full details in the [starter-kit README](starter-kit/README.md).
