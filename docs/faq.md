# FAQ

Anticipated questions about the Three-Body Protocol.

---

### Why "Three-Body"?

Borrowed from physics. Three gravitational bodies have no closed-form solution; you can only simulate, step by step. The metaphor captures the architectural shape of AI-collaborative work — three actors with mutual influence and no shared state. The drama in the name is doing pedagogical work: it reframes coordination friction as the actual structural problem, not as a tooling complaint.

---

### Isn't this just "write things down"?

Yes. The protocol's value isn't novelty — it's specificity. "Write things down" is generic advice. Three required artifacts, with required sections, with a 50-line cap and a foreclosure field, is a discipline you can audit. The shape is what makes it work.

---

### Does this work without Claude Code? With Cursor? With ChatGPT?

Yes. The protocol is about roles, not vendors. Strategic actor + implementing actor + human is the same architecture whether the actors are Claude + Claude Code, Cursor + GPT, Copilot + Gemini, or any future combination. The bridge artifacts are vendor-agnostic.

---

### What if I'm only using one AI?

The protocol scales down. With one AI session and a human, the architecture is two-body, not three-body. STATUS_NOW + DECISIONS_LOG carry most of the bridge; briefs become optional. Don't force the full protocol where it doesn't fit.

---

### What if AI memory features get better?

The bridge load goes down. The protocol still applies — as long as you're using more than one AI session in a workflow, sessions don't share working memory, and the human is the only persistent actor, you have the three-body architecture. Memory features reduce the bridge's role; they don't eliminate it.

---

### Hasn't Anthropic Agent Teams solved this?

Partially. Claude Code Agent Teams (released April 2026) coordinates multiple Claude Code sessions — a lead session and teammates with shared task lists and dependencies. That handles part of what the protocol addresses: implementing-actor-to-implementing-actor coordination at the platform level.

What it doesn't handle is the *strategic-actor case*: when the thinking and planning happens in a chat-window AI (Claude in chat, ChatGPT, etc.) and the implementation happens elsewhere. The strategic actor is outside Agent Teams' coordination surface. Three-Body covers that case.

The two work together. If you use Agent Teams for parallel implementation work, the bridge artifacts (STATUS_NOW, decisions log, briefs) still serve to coordinate between the strategic actor and the team. The team is "the implementing actor" from the protocol's perspective; internal team coordination is Agent Teams' job; cross-actor coordination is still the bridge's job.

---

### How is this different from a project management tool?

PM tools track tasks. The bridge tracks *coordination state for AI sessions* — what an AI session needs to know to pick up where the last session stopped. There's overlap, but the audiences differ. STATUS_NOW is read by an AI agent at session boot. A Jira board isn't.

You can put STATUS_NOW content in a PM tool if you want. The discipline is what matters: 50-line cap, required sections, every session reads it first. The format follows.

---

### Why a 50-line cap?

Empirically: longer files become graveyards. People stop reading them, AI sessions skim past them, the file drifts from reality.

50 lines is the rough threshold where a file is still readable in one screen and updateable in one minute. The exact number isn't sacred — 40 or 60 would work — but a cap is non-negotiable. A STATUS_NOW that grows to 200 lines isn't a status file anymore.

---

### Do I really need the "Forecloses" field in the decisions log?

Yes. It's the field that prevents the most expensive failure mode: re-litigating closed decisions six months later because nobody captured what the decision ruled out. The first three months, this field feels artificial. After six months, it's the field you reach for first when reading old log entries.

---

### What about briefs that go from implementing actor *back* to strategic actor?

Brief dispatch is one-way under v1.0. Return communication is via the result artifacts (the outputs the brief named). If the implementing actor needs to surface a finding or escalate a question, that's a separate brief or a comment in the result artifact, not a modification of the original brief.

A reverse-direction brief format is plausible for v2 if the pattern shows up enough.

---

### How do I version the bridge artifacts?

STATUS_NOW: not versioned — it's current state. Snapshots at session-end are optional but useful for retrospectives.

DECISIONS_LOG: append-only, ordered chronologically. Decisions can be superseded by later decisions (with explicit override entries), but they don't get edited in place.

Briefs: archived with the dispatch artifacts. Not edited after dispatch. Renegotiation produces a new brief.

---

### Can I use this for non-AI work? Just human teams?

You can. The protocol's architecture (multiple actors, no shared memory, asynchronous coordination) maps to distributed teams reasonably well. STATUS_NOW + DECISIONS_LOG + brief format is roughly equivalent to good remote-work hygiene.

But the protocol is *designed* for AI-collaborative work specifically — the failure modes named in the protocol (drift, re-briefing decay, big-bang briefing) are AI-specific in their character even if they have human-team analogs. Adopt at your discretion.

---

### Is this related to "Memex" / "Zettelkasten" / "second brain" / [other note-taking system]?

Adjacent but different. Note-taking systems are designed for personal knowledge management. The bridge artifacts are designed for coordination across actors. There's overlap (the decisions log shares DNA with a Zettelkasten) but the audience and purpose differ. STATUS_NOW is read by *another actor* at session boot; a personal knowledge base is read by you when you remember to.

---

### Where's the implementation?

The protocol is the artifact. There's no central tool. The templates in this repo are copy-paste — drop them into your workspace and start using them.

If interest develops, a thin tool to mechanize the bridge (auto-archive on cap, decision-log validation, brief linting) is plausible for v2. For now: copy, paste, run.

---

### What tools play well with this protocol?

Anything that respects the artifacts. Git is the obvious example — versioning STATUS_NOW and DECISIONS_LOG in git gives you durable history for free. Hooks (pre-commit, session-init) mechanize the discipline. Most other tooling is incidental.

---

### Did you really build this for legal-AI work?

Yes. ORCA's pipeline involves multiple Claude sessions coordinating across spec authoring, code implementation, legal-content drafting, and adversarial review. The three-body problem was acute there because the cost of drift was high — a missed constraint in legal content can mean malpractice exposure. The protocol generalized cleanly to non-legal work, which is why this repo exists.

---

### Suggestions / improvements?

Open an issue on this repo. Real-use feedback is the most valuable input — generic suggestions are less so.
