---
description: Phase 2 — technical design from approved requirements
argument-hint: <feature-name>
---
Read `.claude/specs/$1/requirements.md`. Create `.claude/specs/$1/design.md` with:
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

Before showing me, do a consistency pass: verify every internal cross-reference
resolves (requirement IDs cited in design exist in requirements.md; any section
references point at the section they name), and that every requirement ID
appears in at least one design decision.

STOP and show me. Do not generate tasks until I approve.

Once I approve, prompt me to run the next phase:

> Design approved. Next: `/spec-tasks $1`
