# Three-Body Protocol

Templates for handing work between a planning assistant, a coding assistant,
and a human without relying on conversation memory.

## Start with three files

1. [STATUS_NOW.md](templates/STATUS_NOW.md): the current task, next step, active
   branches, and last actual run. Keep it short; the default cap is 50 lines.
2. [Decision log](templates/decision-log-entry.md): what was decided, why, and
   which alternatives were ruled out.
3. [Brief](templates/brief.md): the task, constraints, source files, expected
   outputs, and acceptance criteria.

Read the status at session start. Check it against the current repository.
Update it when the work ends. Record work that was not tested as untested.

## Example handoff

Synthetic example:

```text
Task: Add a refund endpoint.
Constraint: Repeated requests with the same request_id must not refund twice.
Source: decisions log, entry D-7.
Acceptance: Send the same request twice; only one refund is recorded.
```

The receiving session can verify a specific constraint instead of reconstructing
it from a chat summary. For a fuller example, see the
[decision walkthrough](examples/forecloses-walkthrough.md).

A written handoff can still be stale. Try the
[already-fixed crash test](https://github.com/moranbickel/agent-crash-tests/tree/main/cases/already-fixed)
to see why checking the current code matters before acting on a note.

## Read further

- [Protocol](PROTOCOL.md): session boundaries, roles, archive rules, and review.
- [Adoption guide](docs/how-to-adopt.md) and [session-start checklist](templates/session-start-checklist.md).
- [FAQ](docs/faq.md) and [rationale](docs/rationale.md).

These are working conventions. The files do not enforce compliance, provide
shared live memory, or prove that a decision was correct. Someone still needs
to compare the result with the brief.

Related approaches: [automated decision logs](https://addyosmani.com/blog/automated-decision-logs/)
and [session handoffs](https://github.com/softaworks/agent-toolkit/tree/main/skills/session-handoff).
For branch synchronization, see [Peer-Worker-Convergence](https://github.com/moranbickel/Peer-Worker-Convergence).

Maintained by [Moran Bickel](https://github.com/moranbickel).
Prose: [CC BY 4.0](LICENSE-CC-BY-4.0). Templates and code: [MIT](LICENSE-MIT).
