# ADR-NNNN: Title of the decision

- **Status:** Proposed | Accepted | Superseded by ADR-NNNN
- **Status history:** YYYY-MM-DD Proposed; YYYY-MM-DD Accepted
- **Date:** YYYY-MM-DD (date of the current status)
- **Owner:** the role that answers for this decision
- **Confidence:** high | medium | low, and one sentence why
- **Revisit when:** the observable condition that should reopen this decision
- **Related:** ADR-NNNN (depends on, constrains or is constrained by)

## Context

What problem or requirement forces a decision now? Which constraints apply (people, data, compliance, capacity, budget)? State facts, not the solution.

## Decision

What we decided, in two or three sentences. Someone who reads only this section knows what to do.

## Options considered

1. **Option A.** One or two sentences. *Rejected because* one concrete reason.
2. **Option B.** One or two sentences. *Rejected because* one concrete reason.
3. **Option C (chosen).** One or two sentences.

## Why

The reasons, with links to sources wherever a platform behaviour is the reason. Separate what the platform does (with a source) from what we prefer (labelled as our choice).

## Consequences and trade-offs

What gets easier, what gets harder, what we have to do now because of this decision. Include the costs you accept on purpose; a record that hides its consequences loses its value.

## Sources

Documentation the decision relies on, with the date you checked it.

---

Rules for this log, following [Maintain an architecture decision record (ADR)](https://learn.microsoft.com/azure/well-architected/architect-role/architecture-decision-record?wt.mc_id=AZ-MVP-5003447) in the Azure Well-Architected Framework:

- One decision per record. If a decision has phases (short term, long term), write one record per phase.
- The log is append-only. Don't edit an accepted record; when a decision changes, write a new record that supersedes the old one, and link both. Status and status history are the only lines you update.
- Record the confidence. A decision made with low confidence is the first candidate for review.
- Keep it short and factual. A record is not a design guide; link to longer material instead, but the decision must stand without it.
- Keep the records in Git, next to the code they shape.

The fields *Owner*, *Revisit when* and *Related* are my additions; Microsoft's guidance does not require them.
