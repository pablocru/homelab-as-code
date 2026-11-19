# 0009: ADRs for decisions

## Status

Accepted.

## Context

The repository records its choices in files next to the code they explain, and
those files are read by whoever touches that code later.

## Decision

Short ADRs next to the code explain context, decision, alternatives and
consequences. Every ADR has six sections: Status, Context, Decision, Reasons,
Alternatives discarded and Consequences. Status is `Accepted.` or
`Superseded by NNNN.` A decision changes through a new ADR: the old one's Status
is updated and nothing else in it is rewritten. One decision per ADR and at most
one ADR per pull request.

## Reasons

A decision log lives with the code it explains and does not age like a plan.

## Alternatives discarded

- **Roadmap document:** no durable record of why a decision was made, and stale
  by design.

## Consequences

- A change that needs two decisions becomes two pull requests.
- Superseded ADRs stay in place, so the history of a decision remains readable.
