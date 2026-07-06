# TODOs

Random backlog for the spec kit. Each item is written as a story so it can be
picked up cold. Finished stories move to `TODOs-completed.md`.

## Contents

### Stories

- [Add a `specs/README.md` index (name, type, created, status)](#story-add-a-specsreadmemd-index-name-type-created-status)
  — auto-maintained table giving an at-a-glance view of every spec's type,
  creation date, and status.
- [Surface mid-implementation decisions that deviate from the spec](#story-surface-mid-implementation-decisions-that-deviate-from-the-spec)
  — `/spec-impl` should pause and offer a spec update when a judgment call
  contradicts the design, instead of drifting silently.

### Spikes

- [Package the spec kit as a Claude Code plugin](#spike-package-the-spec-kit-as-a-claude-code-plugin)
  — what plugin packaging takes (hooks, namespacing, distribution); go/no-go
  write-up.
- [Should specs live somewhere other than top-level `specs/`?](#spike-should-specs-live-somewhere-other-than-top-level-specs)
  — weigh `specs/` vs `.claude.specs/` vs `docs/specs/`; recommendation only.

---

## Story: Add a `specs/README.md` index (name, type, created, status)

### Problem

`specs/` folder names are bare feature slugs — deliberately: they're the
identifier typed into every `/spec-*` command, so prefixing them with sequence
numbers or dates (for ordering) would make commands painful to type and the
numbers would rot as priorities shift. But that leaves no at-a-glance view of
what specs exist, when they were created, or where each one stands. Creation
order lives only in git archaeology (`git log --diff-filter=A -- specs/`), and
status (drafted? approved? implemented?) lives nowhere at all — you have to
open each folder and inspect checkboxes. Task-only specs (standalone
`/spec-tasks`, no requirements/design) make it worse: nothing distinguishes
them from full specs without opening the directory.

### What to do

1. Add `specs/README.md` to the starter kit: a one-paragraph orientation
   ("specs are produced by the `/spec-*` commands; see
   `.claude/spec-workflow.md`") plus an index table:

   | Spec          | Type       | Created    | Status            |
   |---------------|------------|------------|-------------------|
   | initial-build | full       | 2026-06-28 | implemented       |
   | export-import | tasks-only | 2026-07-03 | awaiting approval |

   `README.md` (not `INDEX.md`/`STATUS.md`) because code hosts auto-render it
   when browsing `specs/` — the table shows without a click.
2. The row is added by **whichever command creates the spec folder** — not
   just `/spec-requirements`, or task-only specs would never be indexed:
   - `/spec-requirements` adds a `full` row (full-spec flow).
   - `/spec-tasks` adds a `tasks-only` row when running standalone (it
     already detects that mode by checking for `requirements.md`/`design.md`).
3. Status transitions are updated by the phase commands: approval of each
   artifact advances a full spec's status; `/spec-impl` flips either type to
   `implemented` when the last task checkbox is ticked. Keep the status
   vocabulary small — full: `requirements` → `design` → `tasks` →
   `implementing` → `implemented`; tasks-only: `awaiting approval` →
   `implementing` → `implemented`.
4. Any logic that lists existing specs must enumerate **directories only** —
   `specs/README.md` is a file, never a feature. (No name reservation needed;
   feature specs are always folders.)
5. Mirror into passwords-py and backfill rows for `initial-build` and
   `export-import`.
6. Coordination note: this touches the same five command templates as the
   zero-based `$N` substitution story (done — see `TODOs-completed.md`), which
   rewrote them to parse the feature name from `$ARGUMENTS`. Build the index
   rows from that parsed feature name.

### Acceptance criteria

- [ ] Fresh full-spec flow: `/spec-requirements` creates the row (`full`,
      today's date), and each phase approval + `/spec-impl` completion
      advances Status without manual edits.
- [ ] Fresh standalone `/spec-tasks` creates a `tasks-only` row; `/spec-impl`
      completing its last task flips it to `implemented`.
- [ ] `specs/README.md` renders as orientation + table when browsing `specs/`
      on a code host; no command ever treats it as a spec folder.
- [ ] passwords-py has the README with backfilled rows for both existing
      specs, with Created dates taken from git history.

---

## Story: Surface mid-implementation decisions that deviate from the spec

### Problem

During `/spec-impl`, Claude routinely makes judgment calls the spec never
anticipated — picking a library the design didn't name, changing an error-handling
approach because the designed one didn't fit, adding a helper module, tightening
or loosening a requirement's edge case. Today those decisions land silently in
the code. The spec (`requirements.md` / `design.md`) keeps describing the
original plan, so it drifts from reality task by task, and the divergence is
only discovered later — if at all — via `/spec-sync` after the whole
implementation is done. By then the context for *why* each deviation happened
is gone.

### What to do

1. Update `starter-kit/.claude/commands/spec-impl.md` so that when
   implementation requires a decision that contradicts or isn't covered by the
   spec, Claude must:
   - **Stop and tell the user** what the decision is, why the spec's approach
     doesn't hold (or doesn't cover it), and what it plans to do instead.
   - **Ask whether to update the spec** — offer to apply the corresponding edit
     to `requirements.md` / `design.md` (and `tasks.md` if the task breakdown
     is affected) right then, while the reasoning is fresh.
2. Define what counts as a spec-relevant decision (so this doesn't fire on
   every variable name): anything that would change what `/spec-sync` reports —
   deviations from the design's stated approach, new/changed dependencies,
   requirement behavior differences, added or dropped components.
3. If the user declines the spec update, note the deviation in the task's
   completion summary so `/spec-sync` still catches it later.
4. Keep it lightweight: one concise prompt per decision, not a ceremony. If the
   user has said "just implement, don't ask," Claude should batch the
   deviations and report them at the end instead of asking each time.
5. Mirror the change in `starter-kit/.claude/spec-workflow.md` so the guidance
   doc and the command stay consistent.

### Acceptance criteria

- [ ] `spec-impl.md` instructs Claude to pause and ask when a decision
      deviates from or is uncovered by the spec, with the definition of
      "spec-relevant" spelled out.
- [ ] Accepting the offer updates the affected spec file(s) in the same
      session; declining records the deviation for `/spec-sync` to surface.
- [ ] Trivial implementation choices (naming, formatting, internal structure
      that the design doesn't constrain) do **not** trigger a prompt.
- [ ] `spec-workflow.md` documents the behavior.
- [ ] Exercised for real on passwords-py (or the next spec-driven feature):
      at least one deviation prompt observed, spec updated, and `/spec-sync`
      afterwards reports clean.

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
  muscle-memory `/spec-*` names? (Partially answered by the prefix-collision
  spike, now in `TODOs-completed.md`: plugin commands are *always* invoked as
  `/plugin-name:command` with no bare form, so yes — packaging as a plugin
  gives up the bare `/spec-*` names.)
- **Can a plugin ship hooks?** If plugins can contribute hooks without touching
  the host repo's `settings.json`, the transplant problem disappears — verify
  this and how conflicts/ordering work. If not, decide: drop the hooks from the
  plugin, or document a manual merge step.
- Where does `spec-workflow.md` guidance land — can a plugin inject CLAUDE.md
  guidance (skill? auto-loaded context?), or does the host repo still need the
  one-line `@import`?
- Authoring-time DRY: the five command files share a near-identical
  `$ARGUMENTS` parsing preamble (duplicated deliberately — substitution only
  happens on the command file's own content, so no runtime import/reference
  can carry the placeholder; see the zero-based `$N` story). If the kit gains
  a packaging step, should it generate the command files from a shared
  header template plus per-command bodies, giving the preamble one source of
  truth without runtime indirection?
- Distribution: `/plugin marketplace add <org/repo>` from a private GitHub repo
  — what's the minimal setup, and how do updates roll out?
- The specs themselves (`specs/<feature>/`) stay per-repo — confirm nothing in
  the plugin model fights that.

### Outcome

A short write-up (or ADR) with a go/no-go: what the plugin would contain, what
stays repo-local, and the migration path for passwords-py. No implementation.

---

## Spike: Should specs live somewhere other than top-level `specs/`?

### Problem

The kit just moved spec artifacts to a top-level `specs/` directory (see the
DONE story above). But `specs/` is a very generic name for something that is,
in practice, pretty tied to Claude: the files are produced and consumed by the
`/spec-*` commands and their format assumes the Claude workflow. A generic
top-level directory could collide with a project's own notion of "specs"
(RSpec-style test dirs, API specs, OpenAPI files) and doesn't signal its
relationship to the Claude tooling. Something like a `.claude.specs/` sibling
directory would keep the artifacts visually adjacent to `.claude/` (and clearly
Claude-owned) while staying **outside** the `.claude/` permission guard that
caused the original prompting pain.

### Initial thoughts

- The one hard constraint from the previous move must hold: whatever the
  location, it cannot live under `.claude/`, or every `tasks.md` checkbox tick
  prompts again in auto-accept mode. `.claude.specs/` (a sibling, not a child)
  should satisfy this — verify the permission guard matches on the `.claude/`
  path prefix and not something looser like `.claude*`.
- Counterpoint from the DONE story: hidden dotted directories hurt
  discoverability — collapsed in code hosts, ignored by some tools, and specs
  are meant to be reviewer-facing engineering documents. `.claude.specs/` gives
  up some of that win. Is "clearly Claude-owned" worth it?
- Candidate options to weigh: keep `specs/`, `.claude.specs/`, `docs/specs/`,
  or making the location configurable (the DONE story punted on
  configurability — "default to `specs/` and keep it simple unless there's
  demand").
- Whatever the verdict, it feeds the plugin spike: the plugin's commands
  hardcode the path, so this should be settled before packaging.

### Questions to answer

- Do real projects commonly have a conflicting top-level `specs/` already
  (Ruby/RSpec conventions, API spec folders)? How likely is a collision in
  practice?
- Does `.claude.specs/` (or any dot-prefixed sibling) trigger any special
  Claude Code behavior — permission prompts, context loading, ignore rules?
- What do the other SDD tools use (spec-kit, cc-sdd)? Is there an emerging
  convention worth matching?
- Cost of moving again: same file list as the previous migration, plus
  passwords-py. Is the churn justified?

### Outcome

A short recommendation in this file: keep `specs/` or move (and to what), with
the reasoning. If a move wins, write it up as a story mirroring the DONE one
above. No implementation.

---
