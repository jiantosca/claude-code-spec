---
description: Re-sync design + tasks after editing requirements (Kiro-style "sync")
argument-hint: <feature-name>
---
Arguments (feature name):

```text
$ARGUMENTS
```

The first whitespace-delimited token above is the feature name — `<feature>` in
what follows. If the block is empty, stop and ask for the feature name.

The requirements for `<feature>` have changed. Reconcile the downstream
artifacts under `specs/<feature>/`:

1. Re-read requirements.md.
2. Update design.md so it still satisfies every requirement ID. Note what changed.
3. Update tasks.md: add tasks for new/changed requirements, flag tasks that are
   now obsolete (don't silently delete completed work — mark them `~~struck~~`
   with a one-line reason).
4. Consistency pass: re-verify internal cross-references in both files still
   resolve (requirement IDs exist in requirements.md; any section references
   point at the section they name) — editing is when these go stale.

Show me a summary of what changed in each file. STOP before implementing anything.
