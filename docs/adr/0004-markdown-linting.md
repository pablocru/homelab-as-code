# 0004: Markdown linting

## Status

Accepted.

## Context

The repository is documentation-heavy and public, so Markdown is reviewed as
often as code. A linter for it must run in the same places as the rest of the
gate.

## Decision

rumdl is the Markdown linter. The `markdown:lint` task runs it, `lefthook.yml`
runs that task on staged Markdown files, and CI runs it over the whole
repository.

## Reasons

rumdl is a single pinned binary installed by mise, so the check needs no Node
runtime and a broken Python environment cannot disable it. One configuration
file, `rumdl.toml`, holds the rules and the per-file exceptions.

## Alternatives discarded

- **markdownlint-cli2:** needs Node and a second package manager for a check
  that runs on text files.
- **No linter:** formatting drift in a public document is review time spent on
  what a tool can check.

## Consequences

- Every new Markdown file is checked; an exception is declared in `rumdl.toml`
  next to the file name it applies to.
- The rules live in one place, so changing them is a reviewable diff.
