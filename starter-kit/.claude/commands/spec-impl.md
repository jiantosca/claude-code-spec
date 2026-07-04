---
description: Implement spec tasks. Pass a task number for one-at-a-time, or "all".
argument-hint: <feature-name> <task-number | all>
---
Arguments (feature name, then a task number or "all"):

```text
$ARGUMENTS
```

The first whitespace-delimited token above is the feature name — `<feature>` in
what follows; the second, if present, is the task selector. If the block is
empty, stop and ask for the feature name — never read a path with an empty
segment like `specs//tasks.md`.

Read `specs/<feature>/tasks.md`.

Tip (Anthropic-recommended): for larger specs, run this in a FRESH session that
hasn't been used to author the spec — a clean context that just reads the file
produces better implementation than a session cluttered with the design back-and-forth.

If the task selector is a number: implement ONLY that task. Mark it `[x]` when
done, run the relevant tests, then STOP and report. (Kiro's default check-in mode.)

If the task selector is "all": load all tasks into TodoWrite, then implement them
respecting `_Depends:_` order — independent tasks can be delegated to parallel
subagents, dependent tasks wait. Mark each `[x]` as completed and run tests
after each. Report a summary at the end.

If there is NO task selector: find the first unchecked task in the list, show it
to me, and ask whether to run it. Do not start until I confirm.
