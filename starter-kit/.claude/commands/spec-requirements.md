---
description: Phase 1 — generate EARS-format requirements for a feature
argument-hint: <feature-name> <short description>
---
Arguments (feature name, then a short description):

```text
$ARGUMENTS
```

The first whitespace-delimited token above is the feature name — `<feature>` in
what follows; everything after it is the description. If the block is empty,
stop and ask for the feature name — never write to a path with an empty
segment like `specs//requirements.md`.

Author `specs/<feature>/requirements.md` for the described feature.

Work through this in order — do NOT just create the file silently.

## 1. Clarify BEFORE drafting

Read the description and skim the existing codebase first. Then ask me the
questions you genuinely need answered — use the AskUserQuestion tool (batch
related questions; don't drip them one at a time). Focus only on things that
change **which requirements exist**:

- scope / MVP boundaries — what's truly in scope. If the feature is large, prefer
  **"keep all requirements but build in phases"** over cutting scope; only drop
  requirements if I actually want them gone.
- ambiguous, missing, or conflicting behavior
- anything I flagged as an open question or said I'm "open to suggestions" on

Feel free to surface relevant options I didn't ask for (a useful capability the
prompt implies but doesn't mention) — propose them, don't silently assume.

Do NOT ask about pure "how" choices (libraries, algorithms, file formats,
storage mechanism) — those belong to `/spec-design`; capture them as open
questions instead. Do NOT ask what you can answer yourself from the repo. If the
description is genuinely unambiguous and well-scoped, say so and skip to drafting.

## 2. Draft

Write user stories with acceptance criteria in EARS notation:
`WHEN [condition/event] THE SYSTEM SHALL [expected behavior]`.

Cover the happy path, edge cases, and error handling. Structure the document
exactly like this skeleton — story-scoped requirement IDs (`R<story>.<n>`) are
what design and tasks cross-reference, and they let later insertions land as
e.g. R3.9 without renumbering anything:

```markdown
# Requirements: <feature>

## Overview
One short paragraph: what this is and why.

## Domain model (shared vocabulary)
- **Term** — one-line definition, used consistently below.
(Include only when the feature has domain nouns worth pinning down.)

## R1. <Story title>
**User story:** As a <user>, I want <capability>, so that <benefit>.
- R1.1 — WHEN <condition> THE SYSTEM SHALL <behavior>.
- R1.2 — ...

## R2. <Next story>
...

## Non-functional / constraints
- N1 — <cross-cutting constraint: tooling, style, security posture, ...>

## Backlog / future (out of scope)
(Only if scoping decisions parked things worth remembering.)

## Open questions (for design)
1. <genuine "how" decision parked for /spec-design>
```

Park genuine "how" decisions in the **Open questions (for design)** section
rather than inventing answers.

IF I opted into phased delivery, add a one-line **Delivery notes** entry (e.g.
"Built in independently-testable phases") so `/spec-design` knows to leave clean
phase seams. Otherwise omit it — do NOT phase the requirements themselves; they
stay complete and unordered.

## 3. Iterate to approval

Show me the file and explicitly ask whether it's the complete and correct set —
anything missing, wrong, or unwanted — then refine with me until I **explicitly
approve**. Do NOT create `design.md` or `tasks.md` yourself, and do not start
implementing.

## 4. Hand off

Once I approve, state that the requirements are locked and prompt me to run the
next phase (with the actual feature name filled in):

> Requirements approved. Next: `/spec-design <feature>`
