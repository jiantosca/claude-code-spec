---
description: Phase 2 — technical design from approved requirements
argument-hint: <feature-name>
---
Read `.claude/specs/$1/requirements.md`. Create `.claude/specs/$1/design.md` with:
- Architecture overview + a Mermaid diagram
- Components and their interactions, data models, interfaces
- A File Structure Plan (which files change / are created)
- Each design decision tagged with the requirement IDs it satisfies
- IF requirements carry a Delivery-notes phasing hint (or the feature is large):
  structure the architecture with explicit, independently-testable **phase seams**
  and call them out, so the build can ship incrementally

STOP and show me. Do not generate tasks until I approve.

Once I approve, prompt me to run the next phase:

> Design approved. Next: `/spec-tasks $1`
