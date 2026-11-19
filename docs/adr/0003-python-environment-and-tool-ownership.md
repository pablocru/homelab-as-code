# 0003: Python environment and tool ownership

## Status

Accepted.

## Context

Only the Ansible toolchain needs a Python environment, and a public repository
that treats the homelab as production cannot install its packages ad hoc. A Node
project can delegate all its tooling to pnpm, but this one cannot: PyPI does not
officially distribute Biome or Cocogitto and uv has no task runner.

## Decision

`pyproject.toml` declares the Python packages and `uv.lock` pins them with their
transitive dependencies, so the environment is reproducible across machines and
CI. The interpreter version lives in `.python-version`, which uv reads and mise
installs, and `requires-python` keeps it in the series that both Ansible tools
support. mise activates the environment uv owns; `mise.toml` keeps no Python
package of its own.

A tool that must see the project's Python packages, either by importing them or
by sharing a version with one that does, belongs to uv; every other tool belongs
to mise, even when PyPI distributes it. So mise installs the interpreter, the
package manager and every executable the quality gate runs, and uv installs the
environment the project executes in.

## Reasons

`ansible-lint` and `yamllint` are the justified exceptions: `ansible-lint`
imports `ansible-core` and requires `yamllint` as a library, so separating them
would duplicate `ansible-core`, let the versions drift and leave the transitive
set outside a lock.

## Alternatives discarded

- **mise's PyPI backend for the Ansible toolchain:** one virtualenv per package,
  a second `ansible-core` that can drift from the one ansible-lint imports,
  transitive dependencies outside any lock and a `PATH` rewritten at run time.
- **All tooling in uv:** not possible, because the official distributions of
  Biome and Cocogitto are not on PyPI and uv has no task runner.
- **mise `[deps.uv]`:** syncs the environment automatically, but it is
  experimental; an explicit task dependency does the same with stable behavior.

## Consequences

- Adding a Python package means `uv add` and a new `uv.lock`; adding any other
  tool means editing `mise.toml` and refreshing `mise.lock`.
- A task that runs project code depends on the environment sync, which fails
  when `pyproject.toml` changed without a new `uv.lock`.
