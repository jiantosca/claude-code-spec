# TODOs

Random backlog for the spec kit. Each item is written as a story so it can be
picked up cold.

---

## Story: Move spec documents out of `.claude/` to a top-level `specs/` directory

**Status: DONE (2026-07-03).** Note: the "migration note in README" criterion was
dropped by choice — sole user for now, no one to migrate.

### Problem

The kit currently stores spec artifacts (`requirements.md`, `design.md`,
`tasks.md`) in `.claude/specs/<feature>/`. In practice this causes real
friction: Claude Code treats everything under `.claude/` as protected
configuration (settings, permissions, hooks, commands), so **every edit there
prompts for approval even in "Edit automatically" mode**. During `/spec-impl`,
Claude ticks a checkbox in `tasks.md` after every task — which means the user
gets a permission dialog on every single task completion, defeating the point
of auto-accept. (Hit this for real while building passwords-py: every
`- [ ]` → `- [x]` edit prompted.)

### Why moving them is the best practice

- **Specs are engineering documents, not tool config.** Requirements, design,
  and task breakdowns are the plan of record for a feature. They churn
  constantly during implementation and are exactly what a teammate or reviewer
  should find when browsing the repo — not something tucked in a dotted
  tool-config directory.
- **The `.claude/` guard is worth keeping absolute.** The harness prompts on
  `.claude/` edits so a model can't quietly rewrite its own instructions or
  permissions. Allowlisting `Edit(.claude/specs/**)` works, but it punches a
  hole in that invariant and has to be replicated in every user's settings
  (`settings.local.json` isn't committed).
- **Discoverability.** `specs/aws-sso-login/design.md` in a PR diff is
  self-explanatory; `.claude/specs/...` reads like tooling noise and hidden
  directories are collapsed/ignored by default in many tools and code hosts.

### What to do

1. In the starter kit, change the canonical spec location from
   `.claude/specs/<feature>/` to `specs/<feature>/`.
2. Update every hardcoded path reference:
   - `starter-kit/.claude/commands/spec-requirements.md`
   - `starter-kit/.claude/commands/spec-design.md`
   - `starter-kit/.claude/commands/spec-tasks.md`
   - `starter-kit/.claude/commands/spec-impl.md`
   - `starter-kit/.claude/commands/spec-sync.md`
   - `starter-kit/.claude/spec-workflow.md`
   - `starter-kit/README.md` and `starter-kit/MIGRATING-FROM-KIRO.md`
   - `kiro-spec-driven-dev-in-claude-code.md` (narrative doc)
3. Add a short "migrating an existing project" note for repos already using
   `.claude/specs/`: `git mv .claude/specs specs` plus re-copying the updated
   commands (or a sed one-liner over `.claude/commands/spec-*.md`).
4. Consider whether `specs/` vs `docs/specs/` should be configurable; default
   to `specs/` and keep it simple unless there's demand.
5. Migrate passwords-py (the live consumer of the kit) the same way.

### Acceptance criteria

- [x] `grep -r "\.claude/specs" starter-kit/` returns no hits; all references
      point at `specs/<feature>/`.
- [x] Running the full flow (`/spec-requirements` → `/spec-design` →
      `/spec-tasks` → `/spec-impl`) in a fresh project creates and edits files
      only under `specs/<feature>/`.
- [x] With permission mode "Edit automatically", `/spec-impl` completes a task
      — including ticking its checkbox in `tasks.md` — with **zero** permission
      prompts, and edits to `.claude/settings.json`, hooks, or commands still
      prompt.
- [x] README documents the layout. ~~and includes the migration note for
      existing `.claude/specs/` users~~ (skipped — sole user, no migration
      audience).
- [x] passwords-py migrated: specs live in `specs/initial-build/`, the five
      `/spec-*` commands + `spec-workflow.md` updated, and a task checkbox edit
      no longer prompts.

---

## Spike: Package the spec kit as a Claude Code plugin

### Problem

Installing the kit today means copying files into each repo (and passwords-py
already drifted once — the `.claude/specs` → `specs/` move had to be applied in
two places). The starter-kit README already recommends graduating to a plugin
for "one versioned source of truth across many repos" — this spike figures out
what that actually takes.

The sharp edge: **`settings.json` doesn't transplant**. The five commands and
`spec-workflow.md` are all `spec`-prefixed files that drop in cleanly, but a
target repo may already have its own `.claude/settings.json`, so the kit's two
hooks (SessionStart half-install warning, Stop checkbox reminder) can't just be
copied — they'd clobber or have to be hand-merged.

### Questions to answer

- Plugin anatomy: what goes in `.claude-plugin/plugin.json`, how do bundled
  commands get namespaced (`/spec-kit:spec-tasks`?), and does that break the
  muscle-memory `/spec-*` names?
- **Can a plugin ship hooks?** If plugins can contribute hooks without touching
  the host repo's `settings.json`, the transplant problem disappears — verify
  this and how conflicts/ordering work. If not, decide: drop the hooks from the
  plugin, or document a manual merge step.
- Where does `spec-workflow.md` guidance land — can a plugin inject CLAUDE.md
  guidance (skill? auto-loaded context?), or does the host repo still need the
  one-line `@import`?
- Distribution: `/plugin marketplace add <org/repo>` from a private GitHub repo
  — what's the minimal setup, and how do updates roll out?
- The specs themselves (`specs/<feature>/`) stay per-repo — confirm nothing in
  the plugin model fights that.

### Outcome

A short write-up (or ADR) with a go/no-go: what the plugin would contain, what
stays repo-local, and the migration path for passwords-py. No implementation.

---

## Spike: Is the `spec-` command prefix already taken?

### Problem

The kit's public surface is five slash commands named `/spec-requirements`,
`/spec-design`, `/spec-tasks`, `/spec-impl`, `/spec-sync`. Other open-source
spec-driven-development tooling exists (GitHub's spec-kit, cc-sdd's `/kiro-*`
commands, angelsen/claude-kiro, etc.) and more keeps appearing. If a popular
tool claims the same `spec-` command names, installing both in one repo would
collide — and even without a literal collision, sharing a prefix with a
well-known tool invites confusion about whose workflow is running.

### Questions to answer

- Survey the known SDD tools for Claude Code (spec-kit, cc-sdd, claude-kiro,
  anything newer): what slash-command names do they install?
- Does anything ship commands literally named `spec-*`? If yes, which ones
  overlap with ours?
- How does Claude Code resolve duplicate command names (repo vs. user vs.
  plugin, plugin namespacing) — is a collision an error, a shadow, or a pick?
- If there IS a clash: is renaming ours worth it, or does plugin namespacing
  (`/spec-kit:spec-tasks`) make it moot? (Feeds the plugin spike above.)

### Outcome

A short note in this file (or the plugin ADR): list of surveyed tools and their
command names, collision verdict, and a keep/rename recommendation.

---
