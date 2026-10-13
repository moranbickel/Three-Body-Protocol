# Worked Example: A High-Stakes Foreclosure

The [decision-log template](../templates/decision-log-entry.md) shows the *shape* of a `Forecloses` field on a low-stakes decision (which GitHub handle to publish under). This example shows the field doing the work it was built for: carrying a high-consequence decision across the memory gap between sessions, and stopping a well-meaning future session from re-opening a settled, dangerous question.

Everything here is invented for illustration: the service, the dates, and the log text. It is a teaching example, not a record of a real project.

---

## The decision

A small payments service exposes a `POST /refunds` endpoint. Early on, one design choice has outsized stakes: **how does the service guarantee that a refund is paid out at most once?**

The pull toward a simple answer is strong. Clients time out, networks drop, and the obvious fix is "if the call times out, just retry it." But a refund moves real money. A retry whose first attempt actually succeeded pays the customer twice, and a double payout is hard to claw back and shows up on someone's statement.

So the decision is made, and it goes in the log.

```markdown
## 2026-02-10 - Refunds are idempotent, keyed by a client-supplied idempotency key

**Decision:** Every refund request must carry a client-supplied `Idempotency-Key`.
The service stores the key with the first outcome and returns that stored outcome
for any repeat of the same key. A request without a key is rejected.

**Rationale:** Timeouts make duplicate requests unavoidable; the only safe response
is to make a duplicate harmless. A server-generated key cannot do this, because a
client that never saw the first response has no way to repeat the same key. A
double refund is a direct financial loss and is expensive to reverse, so we accept
the cost of requiring every client to generate and persist a key.

**Forecloses:**
- Retry-on-timeout without the key (in clients, in the gateway, or in a queue consumer).
- Batch refund jobs that re-derive keys (for example by hashing order id + amount)
  instead of reusing the key stored with the original request.
- Server-generated keys as a substitute for the client-supplied key.
- An "optional key" mode for legacy callers.

**Related:** 2026-02-08 - Refund state machine: pending -> settled | failed.
```

---

## Three months later - the re-litigation that doesn't happen

A fresh session opens. It has no memory of February. The team is dealing with a wave of refund requests that time out during a provider slowdown, and someone asks the obvious question:

> "Support is asking for a nightly job that finds refunds stuck in `pending` and re-submits them. Can we just derive the key from order id and amount so the job doesn't need the original key?"

This is exactly the **re-briefing decay** the protocol is built around. The session that made the February decision is gone; its reasoning lives only in the log. Without the `Forecloses` field, what happens next is predictable: the new session, reasoning from scratch, finds the nightly job reasonable, and a double-payout path ships as a *feature*. Two refunds for the same order with different amounts would even hash to different keys.

With the field, the cost of detecting the closed decision is one grep:

> The assistant, on boot, reads STATUS_NOW and greps the decisions log for `key`. It finds the 2026-02-10 entry and answers:
>
> "That's foreclosed, and the log says why: batch jobs that re-derive keys are explicitly ruled out, because a re-derived key can differ from the original and a duplicate then pays out twice. The convenience argument was already weighed against the double-payout risk. If you want to revisit it, the protocol's move is an explicit override entry, and the override needs to carry the safeguard that makes the job safe, not just re-assert that it would be easier."

The decision space is *preserved*, not lost. The team doesn't re-argue from zero; it picks up exactly where February left off.

---

## If you do want to revisit it - the override path

Foreclosure is not a straitjacket. A foreclosure can be overridden, but the protocol forces the override to be explicit and to engage the original reason, rather than quietly drifting back to the easy answer. Suppose that months later the service starts persisting the original key on every refund record, so a recovery job can read the exact key it must reuse. The override entry might read:

```markdown
## 2026-08-15 - OVERRIDE of 2026-02-10: recovery job may resubmit using the stored key

**Decision:** Supersedes the batch-job clause of the 2026-02-10 entry. A nightly
recovery job MAY resubmit a stuck refund, but ONLY by reusing the key stored with
the original request. It never derives, hashes, or generates a key.

**Rationale:** The original clause existed because a re-derived key could differ
from the original and cause a second payout. Reading the stored key removes that
path: the resubmission is, by construction, a repeat of the same request. The
convenience benefit the February entry acknowledged can now be taken safely.

**Forecloses:**
- Any recovery path that computes a key instead of reading the stored one.
- Resubmitting refunds whose original key was not persisted; those go to manual review.

**Related:** Overrides part of 2026-02-10 - Refunds are idempotent.
```

Notice what the override is forced to do: it names the *exact reason* the original decision gave, and it ships only because it closes that reason at the mechanism level. An override that just said "the nightly job is fine now, support needs it" would be re-litigation wearing a timestamp. The original entry already weighed that argument. The foreclosure makes the bar for reversal honest.

---

## What to notice

- **The `Forecloses` field is what converts "we decided this once" into "future sessions can't silently undo it."** Without it, the February reasoning evaporates with the session that held it, and a high-stakes question reverts to open.
- **High-stakes decisions are exactly where re-litigation is most dangerous**, and exactly where session memory is least reliable, because the consequential decisions are often the *early* ones, furthest from any current session's context.
- **The foreclosure is cheap to detect (one grep) and the reason travels with it.** A bare "no, we decided against that" would invite "but why?" and re-open the argument anyway. The `Rationale` + `Forecloses` pair answers the why before it's asked.
- **The override path keeps foreclosure from becoming dogma.** A decision can always be reversed, but the protocol forces the reversal to engage the original reason, not bypass it. That keeps a foreclosure revisitable rather than absolute.

---

## See also

- [`templates/decision-log-entry.md`](../templates/decision-log-entry.md) - the entry shape and the "Why Forecloses" rationale.
- [PROTOCOL.md](../PROTOCOL.md) - where the decisions log sits in the bridge.
- [docs/faq.md](../docs/faq.md) - "Do I really need the Forecloses field?"
