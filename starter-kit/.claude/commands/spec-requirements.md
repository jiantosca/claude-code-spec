---
description: Phase 1 — generate EARS-format requirements for a feature
argument-hint: <feature-name> <short description>
---
Author `specs/$1/requirements.md` for the feature: $ARGUMENTS

The feature name is the first whitespace-delimited token of the arguments;
everything after it is the description. If `$1` came through empty, parse the
feature name yourself from the first token of $ARGUMENTS (don't write to a path
with an empty segment like `specs//requirements.md`).

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

Cover the happy path, edge cases, and error handling. Number every requirement
(R1, R2, …) so design and tasks can reference them. Park genuine "how" decisions
in an **Open questions (for design)** section rather than inventing answers.

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
next phase:

> Requirements approved. Next: `/spec-design $1`
