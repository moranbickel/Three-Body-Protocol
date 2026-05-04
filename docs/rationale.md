# Rationale

Why the Three-Body Protocol exists, and the failure modes that produced it.

---

## How I got here

I started ORCA the way most people start an AI-collaborative project: a Claude chat for thinking, and Claude Code for implementation. For the first week, I held the architecture in my head. Felt fine.

By week three, I was making the same drift mistake every other day. The strategic Claude would tell me to do X, with a constraint that we'd discussed in chat. I'd brief Claude Code on X, leave out the constraint because it felt obvious, and Claude Code would ship X-without-the-constraint. Strategic Claude would assess the result, miss that the constraint had been dropped, and recommend a next step that assumed the constraint was honored. By the time I caught it, I'd be three deltas downstream of the divergence point.

By week six, I was opening fresh Claude sessions and re-briefing them from memory. The re-briefs were drifting. Decisions I'd made in week two were getting inverted in week six because I misremembered which way they went. Sessions felt productive. The work was getting worse.

The protocol came from naming the architecture: three actors, two of them AIs, none with shared memory, and me as the fragile bridge. Once the architecture had a name, the fix was obvious — write the bridge down. Stop relying on my memory.

---

## The failure modes the protocol addresses

**Drift.** The strategic actor and the implementing actor accumulate small misalignments because constraints are conveyed via human paraphrase. Each round adds a little more drift. By round four, the actors are working against subtly different versions of the brief. The brief artifact and the explicit-constraints-with-sources discipline kill this.

**Re-briefing decay.** Fresh sessions get briefed from human memory. Human memory is biased toward recent events and reliable-feeling on details that are actually wrong. STATUS_NOW + DECISIONS_LOG replace memory-based briefing with file-based briefing. The new session reads what *was actually decided*, not what the human currently remembers.

**Big-bang briefing.** Either too sparse ("fix the export") or too dense (a fifty-line dump). Both fail. The brief format forces structure: task, inputs, outputs, constraints, out-of-scope, success criteria. The structure is the discipline.

**Re-litigation of closed decisions.** Without a "Forecloses" field in the decisions log, every fresh session asks "have we considered Y?" and re-runs an argument that was settled three months ago. The Forecloses field makes re-litigation cheap to detect.

**Session-end neglect.** Sessions that end without updating STATUS_NOW corrupt the next session's boot. I lost real time to this — opening a session, finding STATUS_NOW out of date, spending fifteen minutes re-deriving where things stood. The session-end protocol is the discipline that closes this gap.

---

## Failure modes the protocol does not address

**Bad scope.** If the human's scope is wrong, the bridge faithfully transmits a wrong scope between the actors. The protocol doesn't fix scope; it fixes coordination *given* scope.

**Weak models.** If the strategic actor is too weak to think well or the implementing actor is too weak to build well, the protocol structures their interaction without improving their output. Capability is upstream.

**Missing domain expertise.** No bridge artifact compensates for actors who don't know the domain. The protocol is for coordination, not for capability.

**Bad inputs.** If the human briefs the strategic actor with wrong assumptions, those assumptions propagate through the bridge to the implementing actor, and the work product is wrong with high fidelity.

**Real-time coordination.** The protocol is asynchronous. Two sessions running simultaneously against the same workspace need additional locking and sequencing; the bridge alone doesn't handle them.

---

## What I'd build differently

**STATUS_NOW from day one.** I introduced it in week six. The first five weeks of drift would have been cheaper to prevent than to recover from.

**Decisions log earlier.** I started one in week eight. The decisions from weeks 1–7 had to be reconstructed from chat history, and several were lost. Don't repeat that. If you start a project today, start the log today.

**Brief format earlier.** I was doing chat-handoff for months before I formalized the brief. Every brief I've sent since the format stabilized has been more reliable than any chat-handoff before it.

**Boot ceremony as a hook.** I enforced it by convention for a long time. Eventually wiring it as a session-init hook removed the failure mode where I'd "just dive in" on a session and skip the boot. Mechanizing the boot ceremony is the highest-ROI 30-minute investment in the whole protocol.

**Cross-model RJ from earlier.** The cross-model audit pattern — using a different model to review the implementing actor's output — caught defects that single-model review missed three rounds in a row. I underused this pattern for a long time. It's the cheapest insurance the protocol has.

---

## The naming choice

"Three-body problem" is borrowed from physics. The metaphor is dramatic on purpose. In physics, three gravitational bodies have no closed-form solution. In AI workflows, the architectural shape is the same — three actors, mutual influence, no shared state — but the math is much easier. Once you have the bridge, the system is tractable.

The drama is doing real work, though. Most "how I use Claude" content treats coordination friction as something to put up with. Naming it as the three-body problem reframes coordination as the *actual* structural issue. That reframe is what makes the rest of the protocol read as architectural rather than tactical.

I was tempted to give it a less dramatic name — "session coordination protocol" or similar. Decided against. Drama in technical naming is fine when the drama is doing pedagogical work, and here it is.

---

*Version notes: this is the v1.0 rationale. Revisions tracked in CHANGELOG.md.*
