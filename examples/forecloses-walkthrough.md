# Worked Example: A High-Stakes Foreclosure

The [decision-log template](../templates/decision-log-entry.md) shows the *shape* of a `Forecloses` field on a low-stakes decision (which GitHub handle to publish under). This example shows the field doing the work it was built for: carrying a high-consequence foreclosure across the memory gap between sessions, and stopping a well-meaning future session from re-opening a settled, dangerous question.

Synthetic in its log text, but the underlying decision is a real one from building [ORCA](../README.md#about-orca). No product internals here — this is an architecture-level principle, the kind of thing the decisions log exists to hold.

---

## The decision

Early in building a legal-AI system, one architectural choice has outsized stakes: **may the system write case-law citations into the court document it generates?**

The pull toward "yes" is strong. A brief with supporting authorities reads as more persuasive; a reviewer who sees citations assumes rigor. But a language model that produces citations will, sometimes, produce citations that do not exist — and a fabricated or misapplied authority in a *filed* document is not a quality defect, it's a sanctionable event. In 2023, U.S. attorneys were sanctioned for filing a brief with case citations a chatbot had invented (*Mata v. Avianca*). That is the failure mode on the table.

So the decision is made, and it goes in the log.

```markdown
## 2026-02-10 — Authorities are QA-only; never written into generated court text

**Decision:** The system may use case law only to *verify* its reasoning during QA.
No case citation is ever inserted into the generated court document. Citations
appear in internal review artifacts, never in the draft that reaches the court.

**Rationale:** A generative model cannot be trusted to cite authorities that exist
and apply. A fabricated or misapplied citation in a filed document is a sanctionable
event, not a quality defect — see the 2023 sanctions for an AI-fabricated-citation
brief (*Mata v. Avianca*). The persuasive value of inline authorities is real but
does not outweigh the downside: one hallucinated cite in one filing is a
catastrophic, irreversible reputational and professional-conduct failure. We get
the verification benefit (QA cross-checks reasoning against real law) without
putting model-generated citations in front of a judge.

**Forecloses:**
- Inserting model-generated case citations into any document that reaches the court.
- "Citation-enrichment" features that add supporting authorities to the draft to
  make it more persuasive.
- Treating a citation that passed QA review as therefore safe to surface in output —
  QA verifies reasoning, it is not a license to print the cite.

**Related:** 2026-02-08 — QA layer verifies reasoning against a curated authority set.
```

---

## Three months later — the re-litigation that doesn't happen

A fresh strategic session opens. It has no memory of February. The operator is thinking about persuasiveness and asks the obvious question:

> "The generated briefs would land harder with a few supporting citations. Can we have the model add the two or three strongest authorities to each argument?"

This is exactly the **re-briefing decay** the protocol is built around. The session that made the February decision is gone; its reasoning lives only in the log. Without the `Forecloses` field, what happens next is predictable: the new session, reasoning from scratch, finds the persuasiveness argument compelling, and the team re-runs the entire February argument — except this time someone might land on "yes," and a sanctions-exposure path ships as a *feature*.

With the field, the cost of detecting the closed decision is one grep:

> Strategic AI, on boot, reads STATUS_NOW and greps DECISIONS_LOG for `citation`. It finds the 2026-02-10 entry and answers:
>
> "That's foreclosed, and the log says why: model-generated citations in court-facing text are a sanctionable-fabrication risk (*Mata v. Avianca* is cited as the precedent for the risk). The persuasiveness argument was already weighed and lost to the downside. If you want to revisit it, the protocol's move is an explicit override entry — and the override needs to carry the safeguard that would actually make inline citations safe, not just re-assert that they'd be nicer to have."

The decision space is *preserved*, not lost. The team doesn't re-argue from zero; it picks up exactly where February left off.

---

## If you do want to revisit it — the override path

Foreclosure is not a straitjacket. A foreclosure can be overridden — but the protocol forces the override to be explicit and to engage the original reason, rather than quietly drifting back to "yes." If, months later, the system gains a verified-citation registry (every citation must resolve against a database of real, current authorities before it can be emitted, or the build fails), the override entry might read:

```markdown
## 2026-08-15 — OVERRIDE of 2026-02-10: inline citations permitted via verified registry

**Decision:** Supersedes the 2026-02-10 foreclosure. Model-generated citations MAY
appear in court-facing text, but ONLY when each cite resolves against the verified
authority registry at emit time; any unresolved cite hard-fails the build and blocks
the document.

**Rationale:** The original foreclosure's reason was fabrication risk, not a
principled objection to citations. The verified registry removes the fabrication
path mechanically — an unverifiable cite cannot be emitted. With the risk closed at
the mechanism level, the persuasiveness benefit the 2026-02-10 entry acknowledged
can now be taken safely.

**Forecloses:**
- Emitting any citation that is not registry-verified at the moment of generation.
- Treating "the model is usually right about citations" as a substitute for the
  mechanical check.

**Related:** Overrides 2026-02-10 — Authorities are QA-only.
```

Notice what the override is forced to do: it names the *exact reason* the original decision gave, and it ships only because it closes that reason at the mechanism level. An override that just said "citations are fine now, they make briefs better" would be re-litigation wearing a timestamp — the original entry already weighed and rejected that argument. The foreclosure makes the bar for reversal honest.

---

## What to notice

- **The `Forecloses` field is what converts "we decided this once" into "future sessions can't silently undo it."** Without it, the February reasoning evaporates with the session that held it, and a high-stakes question reverts to open.
- **High-stakes decisions are exactly where re-litigation is most dangerous** — and exactly where session memory is least reliable, because the consequential decisions are often the *early* ones, furthest from any current session's context.
- **The foreclosure is cheap to detect (one grep) and the reason travels with it.** A bare "no, we decided against that" would invite "but why?" and re-open the argument anyway. The `Rationale` + `Forecloses` pair answers the why before it's asked.
- **The override path keeps foreclosure from becoming dogma.** A decision can always be reversed — but the protocol forces the reversal to engage the original reason, not bypass it. That's the difference between a *decision* and a *taboo*.

---

## See also

- [`templates/decision-log-entry.md`](../templates/decision-log-entry.md) — the entry shape and the "Why Forecloses" rationale.
- [README — DECISIONS_LOG](../README.md#decisions_log) — where the field sits in the bridge.
- [docs/faq.md](../docs/faq.md) — "Do I really need the Forecloses field?"
