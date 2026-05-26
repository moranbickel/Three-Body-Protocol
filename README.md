# Three-Body Protocol

**A coordination protocol for AI-collaborative work involving multiple AI sessions and a human.** Names the architectural problem that emerges when you use one AI for thinking and planning alongside a separate AI agent for editing files and executing work — and gives you the bridge files and session protocols that keep them in sync.

If you've ever spent twenty minutes re-briefing a fresh AI session on what was decided last week, or told a coding agent to do X and watched it do Y because it was missing context, this is the protocol. The pattern shows up across tool combinations — Claude + Claude Code, ChatGPT + Cursor, Copilot + any chat assistant — wherever a "thinking" AI and an "implementing" AI need to stay aligned through a human.

I built it while developing [ORCA](#about-orca), an AI legal reasoning system for Israeli civil litigation. It's part of a series of methodology pieces I'm publishing from that work, alongside [Russian Judge](https://github.com/moranbickel/russian-judge).

---

## The problem

You use Claude in a chat window for thinking, drafting, planning. You use Claude Code in your terminal for editing files and writing code. You're the human who closes the loop between them.

Three actors. Two of them are AIs. None of them share memory.

- The chat-window Claude knows what you discussed in *this* conversation. It has no idea what Claude Code shipped this morning, what decisions you made last week, what your repo actually looks like right now.
- Claude Code knows the state of the repo and what happened in *its* session. It has no idea what the strategic Claude just recommended, what tradeoffs you considered, or why the spec is shaped the way it is.
- You remember everything — for about a week. After that, your memory blurs the same way theirs does, except yours blurs slower and feels reliable when it isn't.

This is the **three-body problem**. The name is borrowed from physics, where three gravitational bodies have no closed-form solution — you can only simulate, step by step. In AI-collaborative work, the math is easier, but the architecture is the same: three actors, mutual influence, no shared state, and any one of them losing the picture corrupts the work of the other two.

Most people don't notice the architecture until something breaks. Then it breaks fast.

---

## What goes wrong

Three failure modes account for almost everything.

### Drift

The strategic Claude tells you to do X. You go to Claude Code and ask it to do X. Claude Code does Y, because it's missing context the strategic Claude had — a constraint that was discussed in chat but never written down, a decision you mentioned in passing twenty messages back, a rule that's "obvious" in the strategic context but invisible at the terminal.

You come back to the strategic Claude with the result of Y. It assumes Y was X. It gives you next-step advice based on a wrong picture. You execute that advice. The drift compounds.

By the third or fourth cycle, the work product is shaped by a sequence of small misalignments that no actor in the system can see, because no actor has the whole picture. Only you do, and only if you're paying attention.

### Re-briefing decay

You start a fresh Claude session next week. It has no memory of last week's session.

You re-brief it from your own memory. Some constraints get dropped because you forgot them. Some decisions get inverted because you misremember which way they went. The session happily proceeds against a degraded version of the true brief.

The tell: every fresh session you open feels like it's starting from a slightly different point than the one before it. That's not paranoia. That's actual drift.

### Big-bang briefing

You hand Claude Code a one-line instruction — "fix the export" — that requires it to infer fifty things you didn't say. Or you hand it a fifty-line dissertation that's exhaustive but disorganized, where the load-bearing constraints are buried among preferences.

Either way, the briefing fails. Either Claude Code makes assumptions you didn't license, or it misses the constraints because they're at the wrong altitude. The result is the same: work that diverges from intent, with the divergence invisible until you read the diff.

---

## The bridge

The fix isn't a better AI. It's the bridge: a small set of artifacts that convert the human's fragile memory into durable, session-readable state.

The bridge is just files. The protocol is a discipline about how those files are written, read, and updated.

### STATUS_NOW

A 50-line living document that captures the current state. Required sections: current task, next step, active state (branches, locks, in-flight work), open backlogs, last run.

The 50-line cap is the discipline. Without a cap, status documents become read-only graveyards — written once, never read, never updated, drifted from reality within a week. The cap forces archival: when STATUS_NOW grows past 50 lines, something gets archived to the decisions log. The cap is what keeps the file actually load-bearing.

Every fresh session opens with `view STATUS_NOW.md`. That's the boot ceremony. No exceptions. The session is now in sync, in seconds.

(This piece names the STATUS_NOW discipline; a standalone deep-dive may follow.)

### DECISIONS_LOG

An append-only file capturing decisions and their rationales. Not a changelog — a *decisions* log. The format anchors on three fields: what was decided, why, and what the decision foreclosed.

The "why" is what makes it durable. A decisions log that records "we picked Postgres" without "because we evaluated four alternatives and X mattered most" is unreadable in three months — by you or by any AI session you brief from it. The "why" is what lets a future reader (human or AI) reconstruct the decision space without re-litigating it.

The "what was foreclosed" matters too. It's how you avoid re-running a closed argument. When a future session asks "have you considered Y?" the log says: yes, we considered Y, and here's why we didn't pick it.

### Briefs

When the strategic AI hands work to the implementing AI, it does so via a brief — a structured document, not a chat message. The brief contains: the task, the constraints (with sources), the inputs the implementing AI will consume, the outputs it will produce, and the success criteria.

A brief is not a conversation. It's a contract. The implementing AI reads it, executes against it, and returns a result that maps back to the success criteria. Anything that wasn't in the brief is either an assumption (which the brief should make explicit) or out of scope (which the brief should also make explicit).

This is the antidote to big-bang briefing. The brief format itself enforces the discipline of separating constraint from preference, inputs from outputs, scope from non-scope.

### Cross-model audit (optional but recommended)

For high-stakes work, dispatch the same artifact to a second AI for adversarial review — typically using a different model than the one that produced it. This is where [Russian Judge](https://github.com/moranbickel/russian-judge) plugs into the three-body protocol.

The point isn't redundancy. It's that different models have different blind spots. A second reviewer with a different baseline catches what the first one missed. In my own work, cross-model review has repeatedly caught defects that same-model review missed, especially after the producing model had already seen the artifact multiple times. The asymmetry between the cost (one extra dispatch) and the benefit (a defect that wouldn't have been caught) is dramatic.

### A short vignette

Suppose you're working on a payment integration. Without the bridge:

> Strategic AI: "Implement the refund endpoint, with idempotency on the request_id."
> Human to coding agent: "implement the refund endpoint."
> Coding agent ships a working refund endpoint, no idempotency check.
> The QA run double-refunds a test transaction in staging.

With the bridge:

> STATUS_NOW names the active constraint: "All payment-mutating endpoints require idempotency keys."
> The brief to the coding agent has, in its Constraints section: "Idempotency on request_id is mandatory; see decisions log entry 2026-04-12."
> Coding agent reads STATUS_NOW and the brief, ships the endpoint with idempotency.
> Russian Judge reviews the diff against the brief's success criteria.

The constraint that would have been dropped in human paraphrase survives because it's written down in artifacts both AIs read.

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

These are defaults, not laws. You can move things around. But decide once, write it down, and refer back to it. The most expensive failures in three-body work are the ones where two of the actors think they own the same decision, or where none of them does.

---

## What this protocol is not

**It's not a replacement for tooling.** Better memory in the AIs themselves — when it arrives — will reduce the bridge's load. But it won't eliminate it. As long as you're using more than one AI session in a workflow, the bridge problem exists. The protocol works whether memory features ship next month or never.

**It's not specific to Claude.** Cursor + ChatGPT + you is the same architecture. Copilot + Gemini + you is the same architecture. Any combination of "thinking AI" + "implementing AI" + "human bridge" produces the three bodies. The protocol is about roles and bridges, not vendors.

**It's not a complete solution.** The protocol structures coordination. It doesn't fix bad inputs, weak models, or unclear scope. It's the floor under your AI-collaborative work, not the ceiling.

---

## Adopt this in 30 minutes

1. **Create a STATUS_NOW.md** in the repo or workspace where the work lives. Use the template from [`templates/STATUS_NOW.md`](./templates/STATUS_NOW.md). Hard-cap at 50 lines.
2. **Create a DECISIONS_LOG.md** alongside it. Append-only. Use the entry format in [`templates/decision-log-entry.md`](./templates/decision-log-entry.md).
3. **Adopt the boot ceremony.** Every session — chat-window or terminal — opens with the strategic Claude or Claude Code reading STATUS_NOW first. Add it as your first instruction in every new session, or wire it as a session-init hook if your tool supports it.
4. **Adopt the brief format** for any non-trivial handoff between strategic AI and implementing AI. Template in [`templates/brief.md`](./templates/brief.md).
5. **Update STATUS_NOW at the end of every session.** Not just before quitting — at the actual end, when you know what's done and what's next. The discipline is that the next session can boot from it cleanly.

The full protocol — including session-end checkpoints, archive rules, and the cross-model audit pattern — is in [`PROTOCOL.md`](./PROTOCOL.md).

---

## Why "Three-Body"

The metaphor is from physics. Three gravitational bodies have no closed-form solution; you can only simulate the system, step by step, accumulating small numerical errors. The problem is famously hard.

The metaphor is dramatic on purpose. In AI-collaborative work, the architectural shape is the same — three mutually-influencing actors with no shared state — but the math is much easier. Once you have the bridge, the system is tractable. You're not solving a chaotic dynamical system; you're just writing things down.

But the drama in the name is doing real work. It signals: *this is an architectural problem, not a tooling complaint.* Most "how I use Claude" content treats coordination friction as something to put up with. The three-body framing names it as the actual structural issue, which is the first step to fixing it.

---

## Related work

After drafting this protocol I found several adjacent pieces. I'm naming each one so the reader can place Three-Body in the existing landscape, not because the differentiation needs to be defended — it's clean, but the credibility of any methodology piece depends on the author having actually surveyed the field.

The closest in spirit is Timothy Rainwater's [multi-agent-coordination-framework](https://github.com/timothyjrainwater-lab/multi-agent-coordination-framework) — a fellow non-technical operator's catalog of patterns from coordinating Claude and GPT on a large test codebase over four months. Rainwater names "Artifact Primacy" ("if it's not in a file, it doesn't exist") and "Protocol Over Memory" as principles, and the overlap with the bridge artifacts here is real. Where Three-Body differs: tighter five-piece scope, the 50-line cap as a structural discipline, the Forecloses field in the decisions log, and explicit coupling to [Russian Judge](https://github.com/moranbickel/russian-judge) as the cross-model audit layer.

Addy Osmani's [Automated Decision Logs in AI-Assisted Coding](https://addyosmani.com/blog/automated-decision-logs/) is the closest decision-log lineage. His piece is conceptual ("teams should capture reasoning"); this protocol shapes it specifically — three required fields, Forecloses to prevent re-litigation, integration with STATUS_NOW.

The [softaworks/agent-toolkit session-handoff](https://github.com/softaworks/agent-toolkit/blob/main/skills/session-handoff/README.md) skill solves a similar coordination problem with a heavier template-per-handoff approach (10 required sections, validation scripts, staleness checks). Three-Body's bridge is lightweight by design — one living STATUS_NOW plus an append-only decisions log, not a context capsule per session. Different shape for different operating tempo.

Anthropic's [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams) (released April 2026) ships native multi-Claude-Code coordination at the platform level. Three-Body addresses the case Agent Teams doesn't: a human strategic actor (chat-window AI + you) coordinating with a separate implementing actor. The two are complementary — bridge artifacts work whether the implementing actor is one Claude Code session or a team of them.

Christian Crumlish's ["Three-AI Orchestra"](https://medium.com/building-piper-morgan/the-three-ai-orchestra-lessons-from-coordinating-multiple-ai-agents-0aeb570e3298) (September 2025) is a sibling framing — three AI agents (Claude Code + Cursor + Claude Opus) coordinated by a human, with per-agent timestamped session logs. Different role assignment (three AIs and a veto-authority human, vs. two AIs and a human-as-persistent-bridge), different metaphor (orchestra, not three-body), but the underlying observation that coordination needs structured artifacts is the same.

---

## Related

This is one of a series of methodology pieces from building [ORCA](#about-orca):

- **[Russian Judge](https://github.com/moranbickel/russian-judge)** — adversarial AI review with structured verdicts. Plugs into Three-Body as the cross-model audit layer.
- **Three-Body Protocol** — *this repo.* Coordination across sessions in time.
- **[Peer-Worker Convergence](https://github.com/moranbickel/peer-worker-convergence)** — coordination across sessions in parallel. The git-convergence layer that complements Three-Body's time-convergence layer.
- **[CSAE](https://github.com/moranbickel/csae)** — attestation chains for AI-generated commits.

More pieces as they're written.

## About ORCA

ORCA — Orchestrated Reasoning for Civil Action — is an AI legal reasoning system I'm building for Israeli civil litigation. It's a decision system, not a document generator: it reasons about which causes of action hold, which elements the evidence supports, and what relief follows. A programmer builds a document generator; a litigator builds a decision system. The system is closed-source; the methodology that produced it is open. This repo publishes the coordination methodology, not ORCA's product internals — no source code, knowledge bases, prompts, customer data, or implementation roadmap.

See my [GitHub profile](https://github.com/moranbickel) for the full body of work and how to follow ORCA's progress.

---

## License

- Prose: [CC BY 4.0](./LICENSE-CC-BY-4.0)
- Templates and code: [MIT](./LICENSE-MIT)

— Moran Bickel
