# DECISIONS_LOG Entry Template

The DECISIONS_LOG is append-only. Every decision with lasting consequence gets an entry. The format is rigid because rigidity is what makes the log readable in three months.

---

## Entry shape

```markdown
## YYYY-MM-DD - <short decision name>

**Decision:** <what was decided, in one sentence>

**Rationale:** <why this and not the alternatives. One paragraph maximum. Cite sources where the decision references a spec, a prior decision, or external evidence.>

**Forecloses:** <what this decision rules out, so future sessions don't re-litigate it. Bullet list, one line each.>

**Related:** <optional. Cross-references to other decisions, by date or short-name.>
```

---

## Worked example

```markdown
## 2026-05-04 - Methodology repos under personal handle, not org

**Decision:** All five methodology repos publish under `github.com/moranbickel/<repo>`,
not under a separate `methodology-public` org.

**Rationale:** The hub is the GitHub profile README; the org would duplicate that
function without adding discoverability. Personal-handle URLs are also more durable
across future career moves than org URLs that may need to be transferred or
abandoned. Trades off slightly less "this is a curated body of work" framing for
substantially less infrastructure.

**Forecloses:**
- Creating a `methodology-public` org for these specific five repos.
- Future repos in the same body of work moving under an org without breaking
  inbound links.

**Related:** 2026-05-03 - Five methodology pieces sequenced, RJ first.
```

---

## What goes in this log

Decisions with lasting consequence. Examples:

- Architectural choices ("we use X, not Y").
- Scope decisions ("this is in, that is out").
- Sequencing decisions ("we ship A before B because...").
- Methodology decisions ("we follow this rule because...").
- Trade-offs ("we accept cost X to get benefit Y").

## What does not go in this log

- Implementation details ("function `foo` returns `Bar`"). Use code comments.
- Status updates ("merged the branch"). Use STATUS_NOW or commit messages.
- Half-decisions in flight ("considering X"). Use STATUS_NOW until decided.
- Notes-to-self that aren't decisions. Use notes files.

## Why "Forecloses"

The most expensive failure mode in long-running projects is re-litigating closed
decisions. Six months later, a new session asks "have we considered Y?" and a
human or AI re-runs the entire argument because nobody captured that Y was
already considered and rejected.

The "Forecloses" field exists to make re-litigation cheap to detect. When future
sessions want to revisit a decision, they grep the log, find the foreclosure, and
either accept it or write an explicit *override* entry that supersedes it. Either
way, the prior decision space is preserved, not lost.
