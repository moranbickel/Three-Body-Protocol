# Three-Body Protocol

**A coordination protocol for work that involves more than one AI session plus a human.** When you use one AI for thinking and planning and a separate AI agent for editing files and writing code, the two drift out of sync. This protocol gives you the shared files and the session habits that keep them aligned.

If you've ever spent twenty minutes re-briefing a fresh AI session on what was decided last week, or told a coding agent to do X and watched it do Y because it was missing context, this is for you. The pattern shows up across tools (Claude plus Claude Code, ChatGPT plus Cursor, Copilot plus any chat assistant), wherever a "thinking" AI and an "implementing" AI have to stay aligned through a human.

I built it while developing [ORCA](#about-orca), an AI legal reasoning system for Israeli civil litigation. It's one of a series of methodology pieces I'm publishing from that work, alongside [Russian Judge](https://github.com/moranbickel/russian-judge).

---

## The problem

You use Claude in a chat window for thinking, drafting, and planning. You use Claude Code in your terminal for editing files and writing code. You're the human who closes the loop between them.

That's three actors. Two are AIs. None of them share memory.

- The chat-window Claude knows what you discussed in *this* conversation. It has no idea what Claude Code shipped this morning, what you decided last week, or what your repo looks like right now.
- Claude Code knows the state of the repo and what happened in *its* session. It has no idea what the strategic Claude just recommended, what tradeoffs you weighed, or why the spec is shaped the way it is.
- You remember everything, for about a week. After that your memory blurs the same way theirs does, except yours blurs more slowly and feels reliable when it isn't.

This is the **three-body problem**. The name comes from physics, where three gravitational bodies have no closed-form solution and you can only simulate the system step by step. In AI-collaborative work the math is much easier, but the shape is the same: three actors, mutual influence, no shared state, and any one of them losing the picture corrupts the work of the other two.

Most people don't notice the structure until something breaks, and by then it's already breaking fast.

---

## What goes wrong

Three failure modes account for almost everything.

### Drift

The strategic Claude tells you to do X. You go to Claude Code and ask for X. Claude Code does Y, because it's missing context the strategic Claude had: a constraint discussed in chat but never written down, a decision you mentioned in passing twenty messages back, a rule that's obvious in the strategic context but invisible at the terminal.

You bring the result of Y back to the strategic Claude. It assumes Y was X. It gives you next-step advice based on a wrong picture. You act on that advice. The drift compounds.

By the third or fourth cycle, the work has been shaped by a string of small misalignments that no actor in the system can see, because no actor has the whole picture. Only you do, and only if you're paying attention.

### Re-briefing decay

You start a fresh Claude session next week. It has no memory of last week's session.

You re-brief it from your own memory. Some constraints get dropped because you forgot them. Some decisions get inverted because you misremember which way they went. The session happily proceeds against a degraded version of the real brief.

The tell: every fresh session feels like it's starting from a slightly different place than the one before. That's not paranoia. It's drift, and you can measure it.

### Big-bang briefing

You hand Claude Code a one-line instruction, "fix the export," that requires it to infer fifty things you didn't say. Or you hand it a fifty-line dissertation that's exhaustive but disorganized, where the load-bearing constraints are buried among preferences.

Either way the briefing fails. Either Claude Code makes assumptions you didn't license, or it misses the constraints because they're at the wrong altitude. The result is the same: work that diverges from intent, and the divergence stays invisible until you read the diff.

---

## The bridge

The fix isn't a better AI. It's the bridge: a small set of files that turn the human's fragile memory into durable, session-readable state.

The bridge is just files. The protocol is a discipline about how those files get written, read, and updated.

### STATUS_NOW

A 50-line living document that captures the current state. Required sections: current task, next step, active state (branches, locks, in-flight work), open backlogs, last run.

The 50-line cap is the discipline. Without a cap, status documents become read-only graveyards: written once, never read, never updated, out of date within a week. The cap forces archival. When STATUS_NOW grows past 50 lines, something gets moved to the decisions log. The cap is what keeps the file worth reading.

Every fresh session opens with `view STATUS_NOW.md`. That's the boot step, no exceptions. The session is in sync in seconds.

(This piece names the STATUS_NOW discipline; a standalone deep-dive may follow.)

### DECISIONS_LOG

An append-only file that captures decisions and their reasons. Not a changelog, a *decisions* log. Each entry has three fields: what was decided, why, and what the decision ruled out.

The "why" is what makes it durable. A decisions log that records "we picked Postgres" without "because we evaluated four alternatives and X mattered most" is unreadable in three months, by you or by any AI session you brief from it. The "why" is what lets a future reader, human or AI, reconstruct the decision without re-arguing it.

The "what it ruled out" matters too. It's how you avoid re-running a closed argument. When a future session asks "have you considered Y?", the log answers: yes, we considered Y, and here's why we didn't pick it.

For a high-stakes case, a foreclosure that carries real consequence and the future session it stops from quietly re-opening it, see [`examples/forecloses-walkthrough.md`](./examples/forecloses-walkthrough.md).

### Briefs

When the strategic AI hands work to the implementing AI, it does so through a brief: a structured document, not a chat message. The brief contains the task, the constraints (with sources), the inputs the implementing AI will read, the outputs it will produce, and the success criteria.

A brief isn't a conversation; it's a contract. But it's a contract the *operator* enforces at review, not one the implementing AI is mechanically held to. The AI can and will drift from it: infer something that wasn't there, skip a constraint, quietly expand scope. Nothing in the protocol stops that mid-flight. What makes the brief a contract is the loop around it. The result gets checked against the success criteria, and a result that doesn't map back is a *detected deviation*, recovered by re-briefing (tighter scope, the missing constraint made explicit), not by accepting the drift. A brief with no review step is only a suggestion; the review is what makes it a contract. Anything that wasn't in the brief is either an assumption the brief should have made explicit, or out of scope the brief should also have named.

This is the antidote to big-bang briefing. The brief format forces you to separate constraint from preference, inputs from outputs, scope from non-scope.

### Cross-model audit (optional but recommended)

For high-stakes work, send the same artifact to a second AI for adversarial review, usually a different model from the one that produced it. This is where [Russian Judge](https://github.com/moranbickel/russian-judge) plugs into the three-body protocol.

The point isn't redundancy. It's that different models have different blind spots, so a second reviewer with a different baseline catches what the first one missed. In my own work, cross-model review has repeatedly caught defects that same-model review missed, especially after the producing model had already seen the artifact several times. The trade is lopsided: one extra dispatch against a defect that would otherwise have shipped.

### A short vignette

Say you're working on a payment integration. Without the bridge:

> Strategic AI: "Implement the refund endpoint, with idempotency on the request_id."
> Human to coding agent: "implement the refund endpoint."
> Coding agent ships a working refund endpoint, no idempotency check.
> The QA run double-refunds a test transaction in staging.

With the bridge:

> STATUS_NOW names the active constraint: "All payment-mutating endpoints require idempotency keys."
> The brief to the coding agent has, in its Constraints section: "Idempotency on request_id is mandatory; see decisions log entry 2026-04-12."
> Coding agent reads STATUS_NOW and the brief, ships the endpoint with idempotency.
> Russian Judge reviews the diff against the brief's success criteria.

The constraint that would have been dropped in a human paraphrase survives, because it's written down where both AIs read it.

---

## Roles, decided once

Coordinating three actors means deciding, before the work starts, who decides what.

| Decision | Owner |
|---|---|
| Scope — what's in, what's out | Human |
| Strategy — how the work is shaped | Strategic AI (with human confirmation) |
| Implementation — how the work is built | Implementing AI |
| Quality floor — when the work ships | Human (with reviewer-AI input via RJ) |
| Sequencing — what gets done in what order | Human |
| Memory — what gets written down, where | Human (this is the bridge) |

These are defaults, not laws. Move things around as you like. But decide once, write it down, and refer back to it. The most expensive failures in three-body work are the ones where two actors think they own the same decision, or where none of them does.

---

## What this protocol is not

**It's not a replacement for tooling.** Better memory inside the AIs, when it arrives, will lighten the bridge's load. It won't remove it. As long as you use more than one AI session in a workflow, the bridge problem exists. The protocol works whether those memory features ship next month or never.

**It's not specific to Claude.** Cursor plus ChatGPT plus you is the same architecture. Copilot plus Gemini plus you is the same architecture. Any "thinking AI" plus "implementing AI" plus "human bridge" gives you the three bodies. The protocol is about roles and bridges, not vendors.

**It's not a complete solution.** The protocol structures coordination. It doesn't fix bad inputs, weak models, or unclear scope. It's the floor under your AI-collaborative work, not the ceiling.

---

## Adopt this in 30 minutes

1. **Create a STATUS_NOW.md** in the repo or workspace where the work lives. Use the template from [`templates/STATUS_NOW.md`](./templates/STATUS_NOW.md). Hard-cap it at 50 lines.
2. **Create a DECISIONS_LOG.md** alongside it. Append-only. Use the entry format in [`templates/decision-log-entry.md`](./templates/decision-log-entry.md).
3. **Adopt the boot step.** Every session, chat-window or terminal, opens by reading STATUS_NOW first. Make it your first instruction in every new session, or wire it as a session-init hook if your tool supports it.
4. **Adopt the brief format** for any non-trivial handoff between the strategic AI and the implementing AI. Template in [`templates/brief.md`](./templates/brief.md).
5. **Update STATUS_NOW at the end of every session.** Not just before quitting, but at the actual end, once you know what's done and what's next. The point is that the next session can boot from it cleanly.

The full protocol, including session-end checkpoints, archive rules, and the cross-model audit pattern, is in [`PROTOCOL.md`](./PROTOCOL.md).

---

## Why "Three-Body"

The metaphor is from physics. Three gravitational bodies have no closed-form solution; you can only simulate the system step by step, accumulating small numerical errors as you go. The problem is famously hard.

In AI-collaborative work the shape is the same, three mutually-influencing actors with no shared state, but the math is far easier. Once you have the bridge, the system is tractable. You're not solving a chaotic system; you're just writing things down.

The drama in the name is doing a job, though. It signals that this is an architectural problem, not a tooling complaint. Most "how I use Claude" writing treats coordination friction as something to put up with. The three-body framing names it as the real structural issue, which is the first step to fixing it.

---

## Related work

I went looking for adjacent pieces after drafting this. I'm naming each one so you can place Three-Body in the existing landscape. The differentiation is clean and doesn't need defending, but any methodology piece is only as credible as the author's having actually surveyed the field.

The closest in spirit is Timothy Rainwater's [multi-agent-coordination-framework](https://github.com/timothyjrainwater-lab/multi-agent-coordination-framework), a fellow non-technical operator's catalog of patterns from coordinating Claude and GPT on a large test codebase over four months. Rainwater names "Artifact Primacy" ("if it's not in a file, it doesn't exist") and "Protocol Over Memory," and the overlap with the bridge artifacts here is real. Three-Body differs in its tighter five-piece scope, the 50-line cap as a structural rule, the "what it ruled out" field in the decisions log, and the explicit coupling to [Russian Judge](https://github.com/moranbickel/russian-judge) as the cross-model audit layer.

Addy Osmani's [Automated Decision Logs in AI-Assisted Coding](https://addyosmani.com/blog/automated-decision-logs/) is the closest decision-log lineage. His piece is conceptual ("teams should capture reasoning"); this protocol gives it a specific shape: three required fields, a ruled-out field to prevent re-litigation, and integration with STATUS_NOW.

The [softaworks/agent-toolkit session-handoff](https://github.com/softaworks/agent-toolkit/blob/main/skills/session-handoff/README.md) skill solves a similar coordination problem with a heavier per-handoff template: ten required sections, validation scripts, staleness checks. Three-Body's bridge is deliberately lighter, one living STATUS_NOW plus an append-only decisions log, rather than a context capsule per session. Different shape for a different operating tempo.

Anthropic's [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams) (released April 2026) ships native multi-Claude-Code coordination at the platform level. Three-Body covers the case Agent Teams doesn't: a human strategic actor (chat-window AI plus you) coordinating with a separate implementing actor. The two are complementary; the bridge artifacts work whether the implementing actor is one Claude Code session or a team of them.

Christian Crumlish's ["Three-AI Orchestra"](https://medium.com/building-piper-morgan/the-three-ai-orchestra-lessons-from-coordinating-multiple-ai-agents-0aeb570e3298) (September 2025) is a sibling framing: three AI agents (Claude Code plus Cursor plus Claude Opus) coordinated by a human, with per-agent timestamped session logs. Different role assignment (three AIs and a veto-holding human, versus two AIs and a human-as-persistent-bridge), different metaphor (orchestra, not three-body), but the same underlying observation that coordination needs structured artifacts.

---

## Related

This is one of a series of methodology pieces from building [ORCA](#about-orca):

- **[Russian Judge](https://github.com/moranbickel/russian-judge)** — adversarial AI review with structured verdicts. Plugs into Three-Body as the cross-model audit layer.
- **Three-Body Protocol** — *this repo.* Coordination across sessions in time.
- **[Peer-Worker Convergence](https://github.com/moranbickel/peer-worker-convergence)** — coordination across sessions in parallel. The git-convergence layer that complements Three-Body's time layer.
- **[CSAE](https://github.com/moranbickel/csae)** — attestation chains for AI-generated commits.
- **[Pre-IMPL Forensic Discipline](https://github.com/moranbickel/Pre-IMPL-Forensic-Discipline)** — catching wrong premises before they become wrong commits (v0.1 draft).

More pieces as they're written.

## About ORCA

ORCA (Orchestrated Reasoning for Civil Action) is an AI legal reasoning system I'm building for Israeli civil litigation. It's a decision system, not a document generator: it reasons about which causes of action hold, which elements the evidence supports, and what relief follows. A programmer builds a document generator; a litigator builds a decision system. The system is closed-source; the methodology behind it is open. This repo publishes the coordination methodology, not ORCA's internals: no source code, knowledge bases, prompts, customer data, or roadmap.

See my [GitHub profile](https://github.com/moranbickel) for the full body of work and how to follow ORCA's progress.

---

## License

- Prose: [CC BY 4.0](./LICENSE-CC-BY-4.0)
- Templates and code: [MIT](./LICENSE-MIT)

— Moran Bickel
