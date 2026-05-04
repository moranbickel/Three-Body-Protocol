# Three-Body Protocol — Specification

**Version:** 1.0
**Status:** Stable
**Author:** Moran Bickel

This document specifies the Three-Body Protocol formally. The README is a friendly entry point; this is the source of truth.

---

## §1. The Three Bodies

The protocol identifies three actors in any AI-collaborative workflow that uses more than one AI session.

**§1.1 The Strategic Actor.** An AI session used for thinking, planning, drafting, scoping, and decision support. Typically a chat-window LLM. Has memory of *its own session* and whatever artifacts it has been shown in that session. Has no memory of other sessions or of the workspace's persistent state unless that state is loaded into the session.

**§1.2 The Implementing Actor.** An AI agent that performs work against the workspace — editing files, writing code, running commands, producing artifacts. Typically a terminal-based agent (Claude Code, Cursor's agent mode, etc.). Has memory of *its own session* and visibility into the workspace at run time. Has no memory of strategic-actor sessions.

**§1.3 The Human Operator.** The only actor with persistent memory across sessions and across actors. Decides scope, sequencing, and quality floor. Owns the bridge (per §3). Is the only point at which the three bodies share state.

**§1.4 Memory boundaries.**
The three bodies do not share working memory. Information that needs to flow between them must flow through artifacts the human controls (per §3).

The protocol assumes session-level memory only. If the AI tools provide cross-session memory features, the protocol still applies — those features reduce the bridge's load but do not eliminate it.

---

## §2. The Bridge Problem

**§2.1 Statement.**
Coordination among the three bodies requires that decisions, state, and intent flow between actors. Because the actors do not share memory, information that is not durably written cannot reliably reach an actor that wasn't present when it was discussed.

**§2.2 The human-memory failure.**
The naive solution — "the human remembers and re-briefs as needed" — fails in practice. Human memory is fragile, biased toward recent events, and unreliable on rationale-and-foreclosure (the *why* behind decisions). Briefs reconstructed from memory drift from briefs written down at the moment of decision.

**§2.3 The cost of un-bridged work.**
Without the bridge, three failure modes emerge: drift between strategic intent and implementation, decay across re-briefed sessions, and big-bang briefing that loses constraints in volume. (See README, "What goes wrong.")

---

## §3. Required Artifacts

The bridge is composed of three artifacts. All three are mandatory under this protocol.

**§3.1 STATUS_NOW.**
A 50-line living document at a known path in the workspace. Required sections:

- **Current task** — what the human is working on right now, in one or two sentences.
- **Next step** — the immediate next action, concrete enough to execute against.
- **Active state** — branches, locks, in-flight work, environments, anything an actor needs to know to avoid stepping on something.
- **Open backlogs** — items in flight or queued, with one-line each.
- **Last run** — timestamp + outcome of the most recent meaningful execution.

Hard cap: 50 lines. The cap is the discipline — without it, the file becomes a graveyard.

When STATUS_NOW grows past 50 lines, the oldest material archives to DECISIONS_LOG (per §3.2) or to a per-task notes file. STATUS_NOW captures *now*; it does not accumulate history.

**§3.2 DECISIONS_LOG.**
An append-only file capturing decisions and their rationales. Each entry has three required fields:

- **Decision** — what was decided, in one sentence.
- **Rationale** — why this and not the alternatives. Brief; one paragraph maximum.
- **Forecloses** — what this decision now rules out, so future sessions don't re-litigate it.

Optional fields: date, related decisions (cross-references), owner.

The entry shape is rigid because the rigidity is what makes the log readable in three months. Free-form decisions logs become unreadable within weeks.

**§3.3 Briefs.**
A brief is a structured document that the strategic actor produces and the implementing actor consumes. Required sections:

- **Task** — what is to be done.
- **Inputs** — what the implementing actor will read, with paths or pointers.
- **Outputs** — what the implementing actor will produce, with paths and acceptance shape.
- **Constraints** — explicit rules the implementing actor must honor; sources cited where the constraint comes from a spec or decision.
- **Out of scope** — what is explicitly *not* part of this brief.
- **Success criteria** — how the human will know the brief was executed correctly.

A brief is not a chat message. It is an artifact. Conversational handoffs between strategic and implementing actors are anti-pattern (per §6.4).

---

## §4. Session Protocols

**§4.1 Boot ceremony.**
Every session — strategic or implementing — begins by reading STATUS_NOW. No exceptions.

For strategic sessions: the human pastes STATUS_NOW into the first message, or links to it, or instructs the session to fetch it. The session does not begin substantive work until it has parsed the file.

For implementing sessions: the agent's first action is `view STATUS_NOW.md` (or the equivalent for the workspace's path). If the file is missing or stale, the session halts and asks before proceeding.

The boot ceremony is what converts re-briefing decay into bounded, deterministic session-start cost.

**§4.2 Mid-session protocol.**
During a session, the actor may update STATUS_NOW only if the change reflects a real state transition (task complete, branch merged, lock released). Cosmetic edits and speculative updates are anti-pattern.

Any decision made mid-session that has lasting consequence is appended to DECISIONS_LOG before the session continues. The discipline is to write the decision *as* it is made, not "later," because "later" is when the rationale gets reconstructed from memory.

**§4.3 Session-end protocol.**
Every session ends with an explicit STATUS_NOW update reflecting the session's terminal state:

- Mark the current task as complete, or update its status.
- Set the next step to the actual next action a successor session will take.
- Note any new active-state items (branches, locks).
- Archive any STATUS_NOW content that has aged past 50 lines.

The session's last act is the STATUS_NOW write. Sessions that end without this discipline corrupt the next session's boot.

**§4.4 Brief dispatch.**
When the strategic actor hands work to the implementing actor, dispatch is via brief (per §3.3), not chat. The brief is committed to a known location (e.g., `docs/briefs/<timestamp>_<task>.md`) before the implementing actor is invoked.

The implementing actor's session opens by reading the brief, executing against it, and returning a result mapped to the success criteria. Anything outside the brief is treated as out-of-scope until renegotiated.

---

## §5. Cross-Model Audit (Recommended)

For high-stakes work, dispatch the implementing actor's output to a *second* AI for adversarial review before accepting it. The reviewer should be a different model from the one that produced the work.

This is where [Russian Judge](https://github.com/moranbickel/russian-judge) plugs into the protocol. RJ's adversarial-review structure (verdict format, C/I/M taxonomy, pass floor) is designed to operate at this seam.

"Different model" admits two interpretations: a different model from the same vendor (e.g., Opus reviewing Sonnet's output), or a different vendor entirely (e.g., GPT reviewing Claude's output). Both catch defects the producer missed. Cross-vendor review tends to catch a class of blind spots — framing assumptions, vendor-specific patterns, evaluation tropes — that same-vendor review can share with the producer. When the work is meta (about LLMs, AI methodology, or AI evaluation itself), the cross-vendor axis is worth the extra setup cost.

The cross-model audit catches blind spots the producing model has on its own output. Single-model RJ catches some defects; cross-model RJ catches more. The cost is one additional dispatch; the benefit is empirically substantial on work that has been seen by the producer multiple times.

The audit is not mandatory under this protocol. It is *recommended* for any work that, if shipped wrong, would cost more than the audit itself.

---

## §6. Anti-Patterns

**§6.1 The "I'll remember" anti-pattern.**
Treating human memory as an adequate bridge. Always fails on a 1–4 week horizon.

**§6.2 The empty-STATUS_NOW anti-pattern.**
Maintaining a STATUS_NOW that has been "kept short" by leaving it empty or near-empty. The cap forces archival, not minimalism. An empty STATUS_NOW is a failure of the protocol, not a success.

**§6.3 The append-everything anti-pattern.**
Treating STATUS_NOW as an append-only log. Violates the cap and turns the file into a graveyard. STATUS_NOW captures *now*; DECISIONS_LOG captures history.

**§6.4 The chat-handoff anti-pattern.**
Strategic actor "tells" implementing actor what to do via paraphrase. The brief format exists because chat handoffs lose constraints, especially the implicit ones the strategic actor was treating as "obvious."

**§6.5 The skip-the-boot anti-pattern.**
Starting a session without reading STATUS_NOW because "I remember where we are." This is the most common protocol failure. The cost of the boot ceremony is small; the cost of skipping it compounds.

**§6.6 The decision-with-no-rationale anti-pattern.**
DECISIONS_LOG entries that record *what* without *why*. Make the log unreadable for any future actor (human or AI) trying to reconstruct the decision space.

**§6.7 The ad-hoc-brief anti-pattern.**
Briefs that omit constraints, success criteria, or out-of-scope sections "because the implementing actor will figure it out." It will figure something out. It will be wrong.

---

## §7. Limitations

**§7.1 Trust in artifacts.**
The protocol assumes the bridge artifacts are trustworthy. If STATUS_NOW or DECISIONS_LOG drift from reality (because someone updated reality without updating the file), the protocol degrades silently. Operator discipline is what keeps the artifacts honest.

**§7.2 Two-actor case.**
The protocol scales down to two actors (one AI + the human) trivially — STATUS_NOW + DECISIONS_LOG carry the whole bridge, and brief dispatch becomes optional. The protocol scales up to four+ actors (multiple strategic AIs, multiple implementing AIs) by adding more brief routing, but the fundamentals don't change.

**§7.3 Real-time coordination.**
The protocol is asynchronous. It does not handle real-time coordination across parallel sessions running simultaneously — that needs additional locking or sequencing discipline that's out of scope here.

**§7.4 What the protocol doesn't fix.**
Bad scope, weak models, unclear inputs, missing domain expertise — these are upstream of coordination. The bridge keeps the actors aligned; it does not improve what the actors produce.

**§7.5 Implementations vary.**
The required artifacts in §3 are the load-bearing minimum, not the maximum. In production deployments, the bridge often acquires additional structure: STATUS_NOW may be implemented as session-block-with-rotation rather than a flat now-snapshot, with archival per session block rather than per line; DECISIONS_LOG may carry session-retrospective entries rather than rigid 3-field decisions, with decisions embedded in narrative; the bridge may have additional layers — project-level backlogs, decision indexes, retro corpora — that are read before STATUS_NOW at session boot. Both shapes are valid. The protocol's promise is that the bridge exists, is durable, and is read by every session — not that any specific file shape is the only legitimate one.

---

## §8. Versioning

This is v1.0. Material changes to the bridge artifacts (adding/removing required files), the role boundaries, or the boot ceremony produce v2.0. Refinements to anti-patterns, terminology, or examples produce v1.1, v1.2, etc.

---

*End of specification.*
