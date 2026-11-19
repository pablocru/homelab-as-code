# 0002: mise as the single interface

## Status

Accepted.

## Context

Every command must work the same way on a developer's machine and in CI. The
control node is Linux today, although a Windows one is possible later through
WSL, so automation must not depend on POSIX shell scripts beyond a task script
that supplies default arguments.

## Decision

Every command is a mise task with a description, so `mise tasks` documents the
project and CI runs the same entry points as a developer. Tasks are named after
their subject and action, never after the tool that implements them, so a name
survives replacing the tool. Toolchain versions live in `mise.toml`; `pin` and
`lockfile` are enabled so additions are exact and `mise.lock` records checksums.
Hooks are mise tasks that call pinned binaries instead of repository-owned
scripts; task scripts only supply default arguments, so hooks also work from Git
Bash or WSL on Windows.

## Reasons

One described entry point per command is what `mise tasks` can document, and
pinning versions in one file keeps a fresh clone and CI on the same toolchain
without a second manifest to maintain.

## Alternatives discarded

- **Shell plus Makefile:** fast to start, but no tool version management and no
  portable hooks; every command would be duplicated in CI.
- **asdf or plain pip and venv:** works, but versions and Python environments
  would be managed twice. mise already wraps both and provides tasks.
- **Download scripts for Biome and Cocogitto:** reimplement what mise already
  does and add repository code to test and maintain.

## Consequences

- Every new command must be a mise task, including the ones CI uses.
- `nodes:ping` is the only task that contacts a node, and it runs outside CI
  because it needs the VPN and `~/.ssh/config`.
- On Windows, Ansible runs inside WSL even though hooks and tasks are portable.
