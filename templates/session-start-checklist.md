# Session-Start Checklist

The boot ceremony, as a checklist. Apply at the start of every session - strategic or implementing.

---

## For implementing actors (Claude Code, Cursor agent, etc.)

```
1. view STATUS_NOW.md
2. If STATUS_NOW.md is missing: HALT. Ask the human before proceeding.
3. If STATUS_NOW.md looks stale - work has moved since its last update
   (new commits, merged branches, or several sessions since the timestamp):
   WARN. Confirm it still reflects reality before proceeding.
4. If a brief is being dispatched: view <brief path>.
5. Confirm: current task, next step, active state (branches, locks).
6. Begin substantive work.
```

## For strategic actors (chat-window Claude, ChatGPT, etc.)

```
1. Paste STATUS_NOW.md into the first message, or instruct the session to fetch
   it (if the session has tool access).
2. If working from a brief or decision context: paste those too.
3. Confirm the session has parsed the state (one-line summary back to you).
4. Begin substantive work.
```

---

## Why each step matters

**Reading STATUS_NOW first.** Boot in seconds, not in twenty minutes of re-briefing.

**Halting on missing.** A session that proceeds without state is a session whose work product can't be trusted. Halting is cheap; recovery from drift is not.

**Warning on stale.** STATUS_NOW that predates real work - commits, merges, decisions made since its last update - may not reflect reality. Measure staleness in intervening *work*, not calendar days: a file untouched across two weeks of no work is fine; one untouched across a heavy afternoon may already be wrong. Better to confirm than to assume.

**Brief read.** If a brief exists, the implementing actor's contract is the brief, not the chat message that pointed at it.

**Confirming the parse.** A session can claim it read the file without actually orienting to it. A one-line summary back ("current task is X, next step is Y") catches this.

---

## Pocket version

> **Read STATUS_NOW. Missing → halt. Stale → confirm before you trust it. Then start.**
