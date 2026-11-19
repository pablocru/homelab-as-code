# 0002: Default branch ruleset

## Status

Accepted.

## Context

`main` holds what runs in the homelab. A direct push, a force push or a merge
that skipped the quality gate would reach the nodes on the next `site:apply`.

## Decision

The default branch only accepts pull requests merged with squash, so history
stays linear, and only when the `Lint` status check passes. The ruleset requires
zero approvals because there is one maintainer, declares no bypass actors, and
blocks deletion and non-fast-forward updates. It is stored in
`.github/rulesets/` and uploaded through the API.

## Reasons

Squash leaves one commit per change on `main`, and that commit is the pull
request title. Requiring `Lint` turns "changes go through a pull request" into
"changes go through a green pull request". Zero approvals and no bypass actors
describe the team as it is: one maintainer, who cannot skip the gate by accident
and can still change the ruleset in Settings.

## Alternatives discarded

- **Classic branch protection:** the older mechanism, configured per branch with
  no file in the repository, so its state is invisible to review.
- **No protection:** one mistake reaches the default branch and the gate becomes
  advisory.

## Consequences

- Direct pushes, force pushes and branch deletion fail for everyone, including
  the maintainer; the escape route is editing the ruleset, not the command line.
- A pull request whose branch is behind `main` must be updated before merging.
- The ruleset file is a record and an input for `gh api`; GitHub does not read
  it on its own.
