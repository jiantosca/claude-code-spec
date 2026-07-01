# claude-code-spec

Kiro-style **spec-driven development** for [Claude Code](https://claude.com/claude-code):
a workflow that turns a feature idea into `requirements.md` → `design.md` →
`tasks.md` and then implements it task-by-task — built entirely from Claude
Code's native primitives (slash commands, `CLAUDE.md`, hooks).

## What's here

### → [`starter-kit/`](starter-kit/) — the kit you actually install
Drop-in `/spec-*` slash commands, the workflow rules Claude follows, optional
hooks, and a worked `example-feature/` spec. Start here if you just want to use
it. See the [starter-kit README](starter-kit/README.md) for install steps, and
[`MIGRATING-FROM-KIRO.md`](starter-kit/MIGRATING-FROM-KIRO.md) if you're coming
from Kiro.

### → [`kiro-spec-driven-dev-in-claude-code.md`](kiro-spec-driven-dev-in-claude-code.md) — the guide
The prose write-up: what Kiro actually does and how to replicate it in Claude
Code. Read this to understand *why* the kit is built the way it is.

### → [`research-findings.md`](research-findings.md) — the raw notes
The verified findings behind the guide (sources, confidence, caveats), preserved
so they can be reused without re-running the original research.

## Quick start

```
Copy starter-kit/.claude/ and starter-kit/CLAUDE.md into your repo,
then run /spec-requirements <feature> in Claude Code.
```

Full details in the [starter-kit README](starter-kit/README.md).
