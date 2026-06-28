---
description: Implement spec tasks. Pass a task number for one-at-a-time, or "all".
argument-hint: <feature-name> <task-number | all>
---
Read `.claude/specs/$1/tasks.md`.

Tip (Anthropic-recommended): for larger specs, run this in a FRESH session that
hasn't been used to author the spec — a clean context that just reads the file
produces better implementation than a session cluttered with the design back-and-forth.

If the second arg is a number: implement ONLY that task. Mark it `[x]` when done,
run the relevant tests, then STOP and report. (Kiro's default check-in mode.)

If the second arg is "all": load all tasks into TodoWrite, then implement them
respecting `_Depends:_` order — independent tasks can be delegated to parallel
subagents, dependent tasks wait. Mark each `[x]` as completed and run tests
after each. Report a summary at the end.
