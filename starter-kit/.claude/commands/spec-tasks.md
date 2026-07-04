---
description: Break a feature into discrete, trackable tasks (uses spec docs if present)
argument-hint: <feature-name> [description, if no spec docs exist]
---
Arguments (feature name, then an optional description):

```text
$ARGUMENTS
```

The first whitespace-delimited token above is the feature name — `<feature>` in
what follows; everything after it is the description. If the block is empty,
stop and ask for the feature name — never write to a path with an empty segment
like `specs//tasks.md`.

Create `specs/<feature>/tasks.md` as a numbered checklist.

First check whether `specs/<feature>/requirements.md` and
`specs/<feature>/design.md` exist:

- If BOTH exist (full-spec mode): base the tasks on them, and tag each task with
  the requirement IDs it implements.
- If they DON'T exist (standalone tasks-only mode): generate tasks directly from
  the description above. Skip the requirement-ID tags.
- If they don't exist AND no description was given: ask me what the feature is
  before writing anything.

For each task include:

- A clear description and expected outcome
- `_Requirements:_` the requirement IDs it implements (full-spec mode only)
- `_Depends:_` task numbers it depends on (blank if independent)

For larger features (or when requirements opted into phased delivery), group tasks
into **phases** — each a working, independently testable slice (own tests,
shippable on its own). Mark each phase boundary with a `_Boundary:_ <phase name>`
line; keep `_Depends:_` for ordering within and across phases. Small features stay
a flat list.

Use `- [ ]` checkboxes. Keep tasks small enough to review in one sitting.
STOP and show me before any implementation.

Once I approve, prompt me to run the next phase (with the actual feature name
filled in):

> Tasks approved. Next: `/spec-impl <feature> 1` (one task at a time) or `/spec-impl <feature> all`.
