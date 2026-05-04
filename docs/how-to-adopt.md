# How to Adopt the Three-Body Protocol

Concrete steps to start using the protocol today. Estimated time to functional adoption: **30 minutes**. Estimated time to fluent use: **two weeks of real work**.

---

## Day one — the 30-minute setup

### Step 1. Create STATUS_NOW.md

In the root of your repo or workspace, copy the template from [`templates/STATUS_NOW.md`](../templates/STATUS_NOW.md). Fill in the required sections with whatever your current state actually is. Don't aim for perfect — aim for accurate. Hard-cap at 50 lines.

### Step 2. Create DECISIONS_LOG.md

Alongside STATUS_NOW. Use the entry format in [`templates/decision-log-entry.md`](../templates/decision-log-entry.md). Add one entry now: the decision to adopt the Three-Body Protocol. This anchors the log with a real entry rather than leaving it empty.

### Step 3. Adopt the boot ceremony

Add a one-liner at the top of your workflow:

> Every Claude Code session opens with `view STATUS_NOW.md`. Every chat-window Claude session opens with the contents of STATUS_NOW pasted into the first message.

For Claude Code: if you have hook support, wire `view STATUS_NOW.md` as a session-init hook. If not, do it manually as your first action — every time, no exceptions.

For chat-window sessions: the cheapest pattern is to keep STATUS_NOW open in a tab and paste it as the first message of every new session.

### Step 4. Adopt the brief format

For your next non-trivial handoff between strategic AI and implementing AI, use the brief template at [`templates/brief.md`](../templates/brief.md). Write it before invoking the implementing actor. Commit it.

### Step 5. End-of-day STATUS_NOW update

Before you close out today's work, update STATUS_NOW. Mark the current task's state. Set next-step to whatever you'll actually do tomorrow. Note any active branches or in-flight work.

That's the loop.

---

## Week one — building the discipline

### Day 1–3: Boot ceremony every time.

You'll forget. That's normal. Notice when you forgot, do the boot ceremony retroactively (paste STATUS_NOW into the chat), and move on. The discipline takes about a week to internalize.

### Day 4–5: Start writing decisions to the log in the moment.

When you make a decision with lasting consequence, write it to DECISIONS_LOG immediately — not "later." "Later" is when the rationale gets reconstructed from memory. The shape (Decision / Rationale / Forecloses) takes a minute to fill in if you do it now and an hour to reconstruct if you don't.

### Day 6–7: Audit your first week.

End of week one: read your STATUS_NOW history (if you've been keeping snapshots) and your DECISIONS_LOG. Look for: decisions that should have been logged but weren't, STATUS_NOW updates that should have happened but didn't, briefs that were chat-shaped instead of file-shaped. Each gap is a habit-formation target for week two.

---

## Week two — fluency

### Mechanize the boot ceremony.

If you haven't already, wire STATUS_NOW reading as a session-init hook for your implementing actor. The cost is 15 minutes; the benefit is removing the "I'll just dive in" failure mode permanently.

### Add the cross-model audit.

For your next high-stakes change, dispatch the implementing actor's output to [Russian Judge](https://github.com/moranbickel/russian-judge) under a different model than the one that produced the work. The first time you see RJ catch something the producer missed, you'll know why this pattern is in the protocol.

### Refine your brief template.

After you've sent 3–5 briefs, you'll notice patterns in your work — sections you always need that aren't in the template, or sections you never use. Edit the template to match your actual work. The template ships in a generic shape; your version should be specific to what you build.

### Start using "Forecloses" actively.

When you make a decision, force yourself to write what it forecloses. The first few times this feels artificial. After a week, you'll notice it changing how you make decisions — surfacing alternatives you hadn't articulated, naming the cost of the path you're choosing.

---

## What "fluent use" looks like

You'll know you've adopted the protocol when:

- You boot every session with STATUS_NOW without thinking about it.
- You write decisions to DECISIONS_LOG in the moment, not later.
- You dispatch briefs as files, not as chat messages.
- A fresh session next month can pick up where today's session ended, without you re-briefing from memory.
- You catch drift earlier — usually within one round, not three.
- You stop opening sessions with twenty minutes of "let me re-explain the project."

This typically takes 2–4 weeks of consistent use.

---

## Common adoption failures

**Skipping the boot ceremony.** The single most common failure. Every time you skip the boot, you re-introduce the architecture problem the protocol exists to solve. Mechanize it (session-init hook) so it can't be skipped.

**Letting STATUS_NOW grow.** The 50-line cap is the discipline. A 200-line STATUS_NOW is a graveyard, not a status file. When the cap is hit, archive — to DECISIONS_LOG (if it's a decision) or to a notes file (if it's not).

**Decisions without rationale.** "Decided X" is half a decision. The Rationale and Forecloses fields are what make the log durable. Without them, you'll re-litigate the same decisions in three months.

**Briefs as chat messages.** The structure is the discipline. A "brief" that's actually three sentences in chat is a chat handoff with a different label.

**Updating STATUS_NOW at session start instead of session end.** STATUS_NOW captures terminal state. Session-end is when you know what's done. Session-start is too late — you've already booted from a possibly-stale file.

---

## When the protocol doesn't fit

Some workflows genuinely don't need it:

- **One-off scripts** where you're only going to run one session ever.
- **Single-actor work** — just you and Claude in chat, no implementing agent. The protocol still applies but the bridge gets lighter (STATUS_NOW + DECISIONS_LOG, no briefs).
- **Real-time pair work** where two humans are coordinating with one AI. The architecture isn't three-body; it's one or two-body.

Don't force the protocol where it doesn't fit. The cost of unnecessary discipline is real.

---

## Need help?

Issues and discussions on this repo are open. If you adopt the protocol in your work and find a sharp edge that the spec doesn't address, I want to know.

— Moran Bickel
