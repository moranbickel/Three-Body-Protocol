# Changelog

## Unreleased

- `examples/forecloses-walkthrough.md` - new high-stakes worked example for the `Forecloses` field (the protocol's most novel feature, previously undemonstrated): a consequential foreclosure, the re-litigation it prevents three months on across the session memory gap, and the explicit override path. Wired into README (DECISIONS_LOG section) and the faq. Closes systemic-review finding S-7.

## v1.3 - 2026-05-04

Pre-publication placeholder substitution.

- All `<handle>` instances substituted to `moranbickel` across README, PROTOCOL, CITATION, how-to-adopt, STATUS_NOW template, and decision-log-entry template.
- All `[last name]` instances substituted to `Bickel` across README, PROTOCOL, CITATION, LICENSE-CC-BY-4.0, LICENSE-MIT, and how-to-adopt.
- `<email>` placeholder removed: About ORCA section's mailto clause dropped; GitHub profile remains as the contact channel (Issues are the primary contact surface for methodology adopters).

## v1.2 - 2026-05-04

Pre-publication prior-art audit (sweeping comparative review against existing methodology pieces).

- README gained a "Related work" section with explicit acknowledgment of: Timothy Rainwater's multi-agent-coordination-framework, Addy Osmani's Automated Decision Logs, softaworks/agent-toolkit session-handoff, Anthropic Claude Code Agent Teams, and Christian Crumlish's "Three-AI Orchestra." Names the differentiation per piece.
- PROTOCOL.md §5 augmented to distinguish same-vendor cross-model audit (e.g., Opus reviewing Sonnet) from cross-vendor cross-model audit (e.g., GPT reviewing Claude). Cross-vendor noted as worth the extra setup cost on meta-LLM work.
- FAQ gained "Hasn't Anthropic Agent Teams solved this?" entry naming Agent Teams (April 2026) as platform-level coordination for implementing-actor-to-implementing-actor; positioning Three-Body as covering the strategic-actor case Agent Teams doesn't address.

## v1.1 - 2026-05-04

Pre-publish refinement based on Code RJ + cross-vendor review.

- README opening made vendor-neutral; explicit cross-tool examples (Claude + Claude Code, ChatGPT + Cursor, Copilot + chat assistant) added.
- README "Cross-model audit" softened: replaced an absolute empirical claim with experience-grounded language.
- README gained a "A short vignette" subsection under "The bridge" - concrete before/after example showing the bridge in action on a payment-integration task.
- README "About ORCA" softened and gained an explicit boundary sentence enumerating what the repo does *not* include (source code, knowledge bases, prompts, customer data, roadmap).
- PROTOCOL.md gained §7.5 "Implementations vary" - names that production deployments may implement STATUS_NOW as session-block-with-rotation and DECISIONS_LOG as session-retrospective rather than the flat shapes in §3.

## v1.0 - 2026-05-04

Initial public release.

- README.md - overview, the three failure modes, the bridge artifacts, adoption pointer.
- PROTOCOL.md - formal specification (§§1-8).
- Templates: STATUS_NOW, decision-log-entry, brief, session-start-checklist.
- Docs: rationale, how-to-adopt, faq.
- Diagram: three-body architecture (`diagram.svg`).
- Licensing: CC BY 4.0 for prose, MIT for templates.
