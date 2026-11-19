# 0004: Quality gate

## Status

Accepted.

## Context

The repository is public and documentation-heavy, so Markdown is reviewed as
often as code. The homelab is treated as production: changes are reviewed,
reproducible and reversible, and the gate that enforces it has to run in the
same places for a developer and for CI.

## Decision

Hooks and CI run the same mise tasks. A single binary, lefthook, configured in
`lefthook.yml`, validates the commit message and runs those tasks, narrowed to
the staged Markdown, JSON and YAML files, while Ansible checks the whole subject
because it needs cross-file context. Hooks are the local, incremental gate; CI
is the shared, complete one. lefthook is installed outside the Python
environment, so a broken environment cannot disable the gate.

rumdl is the Markdown linter: the `markdown:lint` task runs it, `lefthook.yml`
runs that task on staged Markdown files, and CI runs it over the whole
repository. Biome formats and checks JSON wherever it lives.

## Reasons

rumdl is a single pinned binary installed by mise, so the check needs no Node
runtime and a broken Python environment cannot disable it. One configuration
file, `rumdl.toml`, holds the rules and the per-file exceptions.

## Alternatives discarded

- **lefthook inside the Python environment:** ties the quality gate to the
  environment it validates.
- **pre-commit:** Python-centric and hook definitions live in YAML per tool;
  lefthook runs any command directly and is a single binary.
- **Custom hook scripts:** a small script per check works, but duplicates what
  Biome and Cocogitto already do and becomes code to test and maintain. Both
  tools run from mise without adding a Node runtime.
- **Unofficial PyPI wrappers for Biome:** a public repository that treats the
  homelab as production takes each binary from its project's own channel.
- **markdownlint-cli2:** needs Node and a second package manager for a check
  that runs on text files.
- **No Markdown linter:** formatting drift in a public document is review time
  spent on what a tool can check.

## Consequences

- The local gate checks the staged Markdown, JSON and YAML files and runs
  Ansible over the whole subject; CI checks the whole repository.
- `mise install` is the only setup step: its postinstall hook syncs the Python
  environment and installs collections and git hooks, so a clone is fully set up
  once tools are installed.
- Every new Markdown file is checked; an exception is declared in `rumdl.toml`
  next to the file name it applies to.
- The rules live in one place, so changing them is a reviewable diff.
