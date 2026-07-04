# Completed TODOs

Stories from `TODOs.md` that are done, moved here verbatim (with their final
status notes) so the backlog stays scannable.

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

## Story: Zero-based `$N` substitution drops the feature name in the spec commands

**Status: DONE (2026-07-04).** Fixed by dropping positionals — all five
commands now parse the feature name from a fenced `$ARGUMENTS` block. The
passwords-py mirror criterion was dropped by choice (repo left as-is; the
updated kit was exercised end-to-end in the passwords-py-spec-test clone
instead). Follow-on template improvements landed the same day outside this
story's scope: a pinned requirements.md skeleton (story-scoped `R<n>.<m>`
IDs) and a lifecycle/workflow-walk check in spec-design's consistency pass.

> Re-diagnosed 2026-07-04. Originally filed as "Hyphenated feature names break
> `$1` substitution" — the hyphen theory turned out to be wrong; see Findings.

### Problem

The five command files use `$1` for the feature name (e.g. `spec-tasks.md`:
``Create `specs/$1/tasks.md` ``). But Claude Code's positional arguments are
**zero-based**: `$0` is the first argument and `$1` is the *second*
([docs](https://code.claude.com/docs/en/slash-commands)). So `$1` never holds
the feature name — it holds whatever word follows it.

Hit for real in passwords-py (2026-07-03): running
`/spec-tasks export-import export decrypted vault YAML to stdout; ...`
rendered `$1` = `export` — the second whitespace token, which happened to be
identical to the first half of `export-import`, making it look like the
substitution had cut the token at the hyphen (it hadn't; hyphens are never
split). The prompt Claude received literally said `Create
specs/export/tasks.md` and `/spec-impl export 1`; Claude followed it
faithfully and the spec landed in the wrong folder.

Empty positions substitute silently instead of failing, which yields two more
symptoms:

- `/spec-impl` with **no arguments** rendered `Read specs//tasks.md` (observed
  same day) instead of saying "feature name required".
- A command invoked with **only** a feature name (`/spec-design export-import`)
  also gets an empty `$1` — there is no second argument — so even correct
  single-argument invocations build `specs//...` paths.

`$ARGUMENTS` always arrives intact (including `export-import`), so only the
positional expansion is affected. Every command that takes a feature name is
exposed: `spec-requirements`, `spec-design`, `spec-tasks`, `spec-sync` (all
`$1`), and `spec-impl` (`$1` feature + `$2` task number — both off by one).

### Findings (step 1, 2026-07-04)

Characterized in a scratch repo (`~/Projects/claude/claude-command-testing`,
echo commands `/argtest` + `/argtest0`), with the rendered prompts read back
from the session transcripts in `~/.claude/projects/` — i.e. harness output,
not the model's echo. Claude Code 2.1.201:

- **Zero-based indexing, documented — not a bug.** `/argtest one two three` →
  `$1` = `two`; `$0` and `$ARGUMENTS[0]` both give the first argument. Current
  docs: "`$N` — shorthand for `$ARGUMENTS[N]`, such as `$0` for the first
  argument or `$1` for the second."
- **Hyphens are never split.** `/argtest export-import foo bar` → `$1` = `foo`
  (not `export`); `export-import` survives intact in `$0` and `$ARGUMENTS`.
  The original hyphen diagnosis was a coincidence of the incident's wording.
- **Missing positions substitute as empty, silently** — no error, no literal
  `$1` left behind. This is what produces `specs//tasks.md`.
- **Quoting** is shell-style for indexed args (`"hello world"` is one
  argument) but irrelevant here — nothing was being split.
- **No upstream bug to file.** Behavior matches current docs. The ecosystem
  confusion (older docs and blog posts describe shell-style 1-based `$1`) is
  already tracked as anthropics/claude-code#19355, which also notes that
  positional support for *model-invoked* skills is undocumented — one more
  reason not to lean on `$N`.
- **Survey:** GitHub spec-kit's command templates use only `$ARGUMENTS` and
  instruct the model to parse it — no positionals anywhere. cc-sdd's Claude
  templates use `$1` as the feature name (a 1-based assumption), i.e. they are
  broken on current Claude Code exactly the way this kit is.

### What to do

Decision: fix via `$ARGUMENTS` + a parsing instruction — not by renaming
`$1` → `$0`, and not via named arguments (an `arguments: [feature]`
frontmatter list making `$feature` the first argument). Both were considered
and rejected:

- `$N` still substitutes empty silently, so the `specs//tasks.md` failure
  class (no args, single arg) would survive the rename, and the "stop and ask
  when blank" behavior is inexpressible with positionals.
- There is no "rest of the arguments" placeholder, and `spec-requirements` /
  `spec-tasks` take `<feature> <free-text description>` — the model must parse
  "everything after the first token" anyway, so `$0` wouldn't remove the
  parsing instruction, just add a second mechanism beside it.
- The indexing semantics are contested in the wild (docs were 1-based
  historically, #19355 open on the ambiguity, cc-sdd still ships 1-based
  templates); `$ARGUMENTS` + explicit parsing is immune to that churn and
  matches spec-kit's approach.
- Named arguments read better than `$0` but fail the same two tests: a
  missing argument still substitutes empty silently, and a name maps to a
  single position, so the free-text description would still have to be
  parsed out of `$ARGUMENTS` anyway.

1. ~~Characterize the harness behavior first.~~ Done — see Findings above.
2. Make the templates robust: in all five
   `starter-kit/.claude/commands/spec-*.md` files, drop positional `$N`
   entirely. Reference only `$ARGUMENTS` and instruct Claude to take the
   first whitespace-delimited token as the feature name (and, for `spec-impl`,
   the second as the task number / `all`), then derive `specs/<feature>/...`
   paths and the "next command" hints from that. The same instruction handles
   the empty case: if `$ARGUMENTS` is blank, stop and ask for the feature name
   instead of proceeding with an empty path segment. The shape (spec-kit's
   `specify.md` is the reference — it embeds `$ARGUMENTS` in a fenced block
   and parses in-prompt):

   ```markdown
   The first whitespace-delimited token of $ARGUMENTS is the feature name;
   everything after it is the description. If $ARGUMENTS is empty, stop and
   ask for the feature name — never build a path with an empty segment.
   ```

   …then use `specs/<feature>/...` in the body. For the two commands that take
   long free-text descriptions (`spec-requirements`, `spec-tasks`), putting
   `$ARGUMENTS` in its own fenced block keeps the prose from being spliced
   mid-sentence into the template.

   Per-file change inventory (surveyed 2026-07-04):
   - `spec-requirements.md` — `$1` in the target path and the
     `/spec-design $1` handoff hint. Also **remove the existing "if `$1` came
     through empty" fallback**: under 0-based indexing `$1` is the second
     token, which is *non-empty* whenever a description follows, so the guard
     never fires and the path is silently wrong (`specs/build/…`).
   - `spec-design.md` — `$1` in two paths + the `/spec-tasks $1` handoff hint.
   - `spec-tasks.md` — `$1` in three paths + both `/spec-impl $1 …` handoff
     hints.
   - `spec-sync.md` — `$1` twice (prose + path).
   - `spec-impl.md` — `$1` in the tasks path; `/spec-impl export-import 1`
     renders it as `specs/1/tasks.md` (the task number lands in `$1`). The
     body never references `$ARGUMENTS` — it says "the second arg" in prose
     and relies on the harness appending `ARGUMENTS: <raw>` when `$ARGUMENTS`
     is absent — so make the reference explicit while in there. Also define
     the missing-second-token case (feature name but no task number): find
     the next unfinished task in `tasks.md`, present it, and ask the user to
     confirm before executing — neither an error nor an unprompted start.
   - All "next command" handoff hints must emit the *parsed* feature name
     (e.g. "Next: `/spec-design <feature>`"), or an approved phase hands the
     user a broken command.
3. Keep the templates readable — the parsing instruction should be one line,
   not a ceremony, and single-word names must keep working identically.
4. Mirror the updated command files into passwords-py (live consumer), and
   note this as yet another drift-cost data point for the plugin spike below.

### Acceptance criteria

- [x] `/spec-tasks export-import <description>` (unquoted) creates
      `specs/export-import/tasks.md` and its approval message points at
      `/spec-impl export-import ...`. (Verified via scratch smoke tests plus
      three full spec flows with hyphenated `initial-build` in
      passwords-py-spec-test, 2026-07-04.)
- [x] The other four commands resolve a hyphenated feature name to the right
      `specs/<feature>/` paths; `/spec-impl <feature> <n>` still picks up the
      task number correctly.
- [x] `/spec-impl <feature>` with no task number identifies the next
      unchecked task and asks for confirmation before executing it — it
      neither errors nor starts work unprompted.
- [x] A command invoked with only a feature name (`/spec-design
      export-import`) resolves it correctly — no empty path segment from the
      missing second argument.
- [x] Single-word feature names behave exactly as before (`/spec-design
      vault` → `specs/vault/...`).
- [x] Invoking any spec command with no arguments produces a clear "feature
      name required" response — never a path built from an empty segment
      (`specs//tasks.md`).
- [x] Findings from step 1 (tokenization rules, whether quoting helps, bug or
      not) recorded in this story (2026-07-04). Not a harness bug — zero-based
      indexing is documented — so no upstream issue filed; the docs ambiguity
      is already tracked as anthropics/claude-code#19355.
- [x] ~~passwords-py's `.claude/commands/spec-*.md` updated to match~~
      (dropped by choice — passwords-py is being left as-is for now; the
      updated kit was exercised in the passwords-py-spec-test clone instead).

---
