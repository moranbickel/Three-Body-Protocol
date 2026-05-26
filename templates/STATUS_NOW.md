# STATUS_NOW Template

A minimal STATUS_NOW for adoption today. The dedicated [`status-now-discipline`](https://github.com/moranbickel/status-now-discipline) repo (coming) goes deeper on the discipline.

**Cap the length. 50 lines is my default** — and the skeleton below already uses ~30, so that leaves ~20 for live content. The *cap* is the protocol, not the specific number: tune it to your sections, but keep it small enough that the file forces archival. An uncapped status doc rots into a read-only graveyard — that's the failure the cap prevents.

---

Copy the template below into `STATUS_NOW.md` at the root of your repo or workspace. Update at session start (boot ceremony) and session end. Keep it under 50 lines.

```markdown
# STATUS_NOW

**Last updated:** <ISO timestamp>
**Updated by:** <session label or human>

## Current task

<one or two sentences. What you are working on right now.>

## Next step

<the immediate next action. Concrete enough that the next session can execute against it without re-deriving context.>

## Active state

- Branch: <current branch>
- Locks: <any in-flight work that another session shouldn't touch>
- Environment: <test / staging / prod / local — only if relevant>
- Other: <session-specific state that another session needs to know>

## Open backlogs

- <one-line item> — <one-line status>
- <one-line item> — <one-line status>
- <one-line item> — <one-line status>

## Last run

- **When:** <ISO timestamp>
- **What:** <command, dispatch, deploy, etc.>
- **Outcome:** <PASS / FAIL / PARTIAL — and one line of detail>
```

---

## Update discipline

- **Boot ceremony:** every session reads STATUS_NOW first. No exceptions.
- **Mid-session:** update only on real state transitions (task complete, branch merged, lock released).
- **Session-end:** the session's last act is the STATUS_NOW write. Mark current-task status, set next-step to a real next action, archive anything that pushes the file past 50 lines.

## When STATUS_NOW exceeds 50 lines

Archive the oldest material. Two clean targets:

- Decisions with rationale → `DECISIONS_LOG.md` (one entry per decision)
- Notes that aren't decisions → `notes/<task>.md` or similar

Having *a* cap is non-negotiable; the exact number is yours. A STATUS_NOW that's allowed to grow without one stops being load-bearing — nothing forces the archival that keeps it current.
