# 0011: Amending an accepted ADR

## Status

Accepted.

## Context

An accepted ADR states facts as well as decisions: which tasks reach a node,
which tool owns a dependency, which files exist. A later ADR can make one of
those statements false while its decision stays in force. Rewriting the record
in silence hides why the text changed; superseding the whole ADR claims a
decision was reversed; leaving the statement leaves the repository asserting
something untrue, and the next reader has to guess which of the two is current.

The rule that lets the first pull request correct the record exists only once
that pull request has written it, so the pull request carries two ADRs and
declares the exception its own rule grants.

## Decision

A statement of fact in an accepted ADR that a later decision makes false is
corrected in place, in the same pull request as the change that made it false.

- Only the statements the later decision invalidated change. The decision, the
  reasons, the alternatives and every other consequence stay as written.
- Status records it as `Accepted. Amended by NNNN.`, where NNNN is the ADR whose
  decision made the statement false, so a reader follows the record forward.
- An amendment is not a supersession. `Superseded by NNNN.` keeps meaning that
  the decision of that ADR no longer applies; there is no partial supersession.
- A pull request carries a second ADR when that ADR is what keeps an accepted
  ADR true, and its description says which rule it is bending.

## Reasons

A reader opens an ADR to learn what the repository does now, so the current text
has to be true; the Status line and Git blame keep the change visible for anyone
who wants the wording that was replaced.

## Alternatives discarded

- **Superseding the whole ADR:** says a decision was reversed when only a fact
  changed, and leaves no valid record of the decision that still holds.
- **Partial supersession:** the false statement stays in the body, so a reader
  who stops at Consequences still reads it, and the status ends up describing a
  document instead of a decision.
- **Leaving the text and pointing at the newer ADR:** works only for a reader
  who reads both, and nothing tells them they must.
- **Correcting in silence:** loses the reason and the review, because the change
  is invisible in the record itself.

## Consequences

- An amendment commit corrects one ADR and names the ADR that caused it.
- A pull request may carry two ADRs when the second keeps the record true; the
  exception is declared in its description, not inferred.
- Wording that was replaced survives only in Git history.
