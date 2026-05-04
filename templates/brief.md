# Brief Template

A brief is the structured handoff from the strategic actor (chat-window AI + human) to the implementing actor (terminal agent). It is an artifact, not a chat message. It gets written to a file before the implementing actor is invoked.

---

## Required structure

```markdown
# Brief: <short task name>

**Date:** <YYYY-MM-DD>
**Author:** <strategic actor or human>
**Implementing actor:** <Claude Code, Cursor, etc.>

## Task

<One paragraph. What is to be done, at the level of intent.>

## Inputs

- <file or pointer> — <one-line description>
- <file or pointer> — <one-line description>
- <upstream decision or spec> — <one-line description with link>

## Outputs

- <file path> — <expected shape>
- <file path> — <expected shape>
- <commit / PR / verdict> — <expected shape>

## Constraints

- <constraint, with source> — e.g., "Must preserve interface X (see docs/spec.md §4)"
- <constraint, with source>
- <constraint, with source>

## Out of scope

- <explicitly excluded item>
- <explicitly excluded item>

## Success criteria

- <how the human will know the brief was executed correctly>
- <observable signal: tests pass, file exists with expected shape, etc.>
```

---

## Why each section is mandatory

**Task** — without it, the implementing actor optimizes against the wrong goal. "Implement X" without intent ("so that Y" or "to fix Z") produces work that ticks the box but misses the point.

**Inputs** — without it, the implementing actor either reads everything (slow, noisy) or guesses what to read (drift). Explicit input lists scope the actor's read-set.

**Outputs** — without it, the implementing actor's verdict on "done" is unverifiable. Explicit output paths and shapes turn "done" into a binary, mechanical check.

**Constraints with sources** — without sources, the implementing actor treats constraints as preferences and overrides them under load. Sources make constraints traceable to the spec or decision they came from, which is what makes them durable across rounds.

**Out of scope** — without it, the implementing actor scope-creeps. Either it does too much (and breaks adjacent things) or it does adjacent things "while it's there." Explicit out-of-scope sections close the gap.

**Success criteria** — without it, the implementing actor self-asserts done-ness. Explicit success criteria let the human (and any reviewer AI like Russian Judge) verify the work product against an external standard.

---

## Worked example

```markdown
# Brief: Migrate STATUS_NOW boot to a session-init hook

**Date:** 2026-05-04
**Author:** strategic Claude + Moran
**Implementing actor:** Claude Code

## Task

Wire the STATUS_NOW boot ceremony as an automatic session-init hook so that every
implementing-actor session reads STATUS_NOW.md as its first action without manual
prompting. Currently the boot is enforced by convention; this brief makes it
mechanical.

## Inputs

- `.claude/hooks/session-start.sh` — existing hook file, currently empty
- `STATUS_NOW.md` — the file to be read at session start
- Existing convention documented in `docs/methodology/three-body-protocol.md`

## Outputs

- `.claude/hooks/session-start.sh` — populated with the boot logic
- `tests/test_session_start_hook.sh` — smoke test that the hook runs and exits 0
- A test run showing the hook fires on a new session

## Constraints

- Hook must exit 0 even if STATUS_NOW.md is missing (with a warning to stderr) —
  per Three-Body Protocol §4.1, missing STATUS_NOW halts the *substantive* work,
  not the hook itself.
- Hook must respect `LC_ALL=C.UTF-8` and `command -v` guards (see hook-hygiene
  rule in `docs/conventions/hooks.md`).
- No external network calls in the hook.

## Out of scope

- Auto-updating STATUS_NOW (separate brief).
- Hooking strategic-actor sessions (chat-window) — those use a manual paste pattern.
- Cross-session locking.

## Success criteria

- New Claude Code session opened in this repo prints STATUS_NOW.md content as
  its first action.
- `tests/test_session_start_hook.sh` exits 0 on a fresh checkout.
- Hook handles a missing STATUS_NOW.md gracefully (warning, exit 0).
```

---

## Brief lifecycle

1. **Write the brief** to a known location (e.g., `docs/briefs/<timestamp>_<task>.md`).
2. **Commit the brief** before invoking the implementing actor. The brief is part of the audit trail.
3. **Implementing actor reads the brief first**, executes against it, and reports back mapped to the success criteria.
4. **The result is verified against the success criteria** by the human, or by a reviewer AI (Russian Judge), or both.
5. **The brief is archived** with the result. It does not get rewritten retroactively.

---

## Anti-patterns

- **Briefs as chat messages.** Lose constraints, especially implicit ones. The format exists to surface what chat hides.
- **Briefs without success criteria.** Self-asserted "done" by the implementing actor.
- **Briefs without out-of-scope sections.** Scope creep is the default behavior of capable agents; out-of-scope sections are the discipline that prevents it.
- **Briefs that grow during execution.** A brief is a contract at dispatch. Mid-flight changes are renegotiation, which produces a *new* brief, not edits to the old one.
