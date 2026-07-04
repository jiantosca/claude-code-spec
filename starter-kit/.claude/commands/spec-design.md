---
description: Phase 2 — technical design from approved requirements
argument-hint: <feature-name>
---
Arguments (feature name):

```text
$ARGUMENTS
```

The first whitespace-delimited token above is the feature name — `<feature>` in
what follows. If the block is empty, stop and ask for the feature name — never
read or write a path with an empty segment like `specs//design.md`.

Read `specs/<feature>/requirements.md`. Create `specs/<feature>/design.md` with:

- Architecture overview + a Mermaid diagram
- Components and their interactions, data models, interfaces
- A File Structure Plan (which files change / are created)
- Each design decision tagged with the requirement IDs it satisfies
- Cross-reference by stable ID (requirement IDs like R1.2, decision IDs like D4)
  or by section *name* ("the File Structure Plan") — never by section number
  (§5), which goes stale when sections are added or reordered during drafting
- IF requirements carry a Delivery-notes phasing hint (or the feature is large):
  structure the architecture with explicit, independently-testable **phase seams**
  and call them out, so the build can ship incrementally

Before showing me, do a consistency pass:

- Verify every internal cross-reference resolves (requirement IDs cited in
  design exist in requirements.md; any section references point at the section
  they name), and that every requirement ID appears in at least one design
  decision.
- Walk each major operation from the requirements (create, read, update,
  delete, import, …) end-to-end through the named components, and confirm
  every domain object has a defined lifecycle — in particular, who *creates*
  it and how. Decisions that work in isolation can still collide (e.g. a
  design that only defines objects as views over existing data, plus a
  command that must create new ones); fix the collision before showing me.

STOP and show me. Do not generate tasks until I approve.

Once I approve, prompt me to run the next phase (with the actual feature name
filled in):

> Design approved. Next: `/spec-tasks <feature>`
