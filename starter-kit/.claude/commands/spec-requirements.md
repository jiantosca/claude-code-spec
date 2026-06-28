---
description: Phase 1 — generate EARS-format requirements for a feature
argument-hint: <feature-name> <short description>
---
Create `.claude/specs/$1/requirements.md` for the feature described as: $ARGUMENTS

Write user stories with acceptance criteria in EARS notation:
`WHEN [condition/event] THE SYSTEM SHALL [expected behavior]`.

Cover the happy path, edge cases, and error handling. Number every requirement
(R1, R2, …) so design and tasks can reference them.

When done, STOP and show me the file. Do not proceed to design until I approve.
