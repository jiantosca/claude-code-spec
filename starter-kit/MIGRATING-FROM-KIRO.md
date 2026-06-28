# Migrating from Kiro to Claude Code

For developers who already use Kiro's spec-driven development at work and are
picking up Claude Code. The methodology is the same; the mechanics differ. This
page maps what you know onto what's here.

## The one-sentence version

Kiro's phase **buttons** become **slash commands**, Kiro's **steering** becomes
**CLAUDE.md**, and your specs still live in the repo as `requirements.md` →
`design.md` → `tasks.md`. Everything is plain markdown + git.

## Side-by-side

| In Kiro you… | In Claude Code you… |
|---|---|
| Click **Requirements** to generate `requirements.md` | Type `/spec-requirements <feature> <description>` |
| Click **Design** | Type `/spec-design <feature>` |
| Click **Tasks** | Type `/spec-tasks <feature>` (or `/spec-tasks <feature> <description>` for a standalone task list, no full spec) |
| Approve each phase before the next | Same — each command stops and waits for you |
| Click a single task to run it | `/spec-impl <feature> <task#>` |
| Click **Run all Tasks** | `/spec-impl <feature> all` |
| Edit requirements, Kiro re-syncs design/tasks | `/spec-sync <feature>` |
| Rely on steering files for project rules | Put rules in `CLAUDE.md` |
| Find specs in `.kiro/specs/<feature>/` | Find them in `.claude/specs/<feature>/` |

## Behavioral differences worth knowing

These are the places it does **not** behave identically — read them so nothing
surprises you.

1. **Gates are prompt-driven, not UI-enforced.** Kiro's UI hard-stops between
   phases. Here, each command *instructs* Claude to stop and wait for approval.
   It's reliable in practice, but it's a convention enforced by the prompt (and
   optionally a hook), not a button you can't click past. You can always just
   tell Claude to keep going.

2. **No isolated-context wave scheduler.** Kiro's "Run all Tasks" groups
   independent tasks into concurrent waves, each in its own isolated context.
   `/spec-impl all` reproduces the *dependency ordering* (via the `_Depends:_`
   annotations) and can fan independent tasks out to subagents, but it is not a
   byte-for-byte copy of Kiro's wave engine. For maximum control, prefer
   one-task-at-a-time (`/spec-impl <feature> <n>`) — which is how Kiro worked
   originally, and by design.

3. **Specs live under `.claude/specs/`, not `.kiro/specs/`.** A deliberate choice
   to keep all Claude Code artifacts together. If you want byte-compatibility with
   Kiro specs (e.g. to open the same files in both tools), change the path in the
   five command files and in `.claude/spec-workflow.md` to `.kiro/specs/`.

4. **EARS is convention, not a feature.** The `WHEN … THE SYSTEM SHALL …` format
   is enforced by the requirements command's prompt — nothing technical depends on
   it. Keep it for testability, or relax it if your team prefers.

5. **Version control is just git.** Kiro has no special VC handling for specs and
   neither does this — the files are markdown you commit like any other. Committing
   `.claude/commands/` means the *method itself* travels with the repo, so the
   whole team gets the same `/spec-*` commands automatically.

6. **Plan mode is a bonus you didn't have in Kiro.** Press `shift+tab` to cycle
   into plan mode during the requirements/design phases — Claude proposes without
   touching files until you accept.

7. **Don't spec everything — right-size the ceremony.** Just as you don't always
   create a full spec in Kiro (sometimes just a tasks.md, often just vibe-coding a
   small fix), the same applies here, and Anthropic's own guidance agrees: *"if you
   could describe the diff in one sentence, skip the plan."* The ladder:
   - **one-sentence diff** → just ask Claude (vibe code)
   - **uncertain / multi-file, one sitting** → plan mode (`shift+tab`)
   - **multi-step, want a durable checklist** → `/spec-tasks <feature> <desc>` standalone
   - **big / shared / multi-session feature** → the full `/spec-*` flow

8. **Native "Tasks" is not the same as `tasks.md`.** Claude Code has a built-in
   **Tasks** primitive (the `TaskCreate`/`TaskUpdate` tools) that tracks live
   progress and now persists across sessions and subagents. But it's **machine-local
   and never uploaded** — it does *not* travel in git, so your teammates don't see
   it. This kit's `tasks.md` is the opposite: a committed, reviewable plan in the
   repo. Don't confuse the two: native Tasks = your local progress tracker;
   `tasks.md` = the shared, version-controlled spec artifact. They work together.

## Your first spec in 60 seconds

```
/spec-requirements search-filters  let users filter the results list by date and status
# review requirements.md, tweak if needed, approve

/spec-design search-filters
# review design.md (architecture + Mermaid + file plan), approve

/spec-tasks search-filters
# review tasks.md (numbered, with dependencies), approve

/spec-impl search-filters 1
# Claude implements task 1, runs tests, stops. Repeat: 2, 3, …
# or once you trust the plan: /spec-impl search-filters all
```

That's the whole loop. If you change your mind on requirements mid-stream, edit
`requirements.md` and run `/spec-sync search-filters` to reconcile design and
tasks.

## See also

- `README.md` — install and full command reference
- `.claude/specs/example-feature/` — a complete worked example (dark-mode toggle)
- `.claude/spec-workflow.md` — the workflow rules Claude follows in every session
  (imported by `CLAUDE.md`)
