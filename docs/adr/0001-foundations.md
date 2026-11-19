# 0001: Foundations

## Status

Accepted.

Foundational snapshot: it records several decisions, each of which may be
superseded in part by a later ADR.

## Context

`homelab-as-code` starts from an empty repository. It manages a personal homelab
from a Linux control node, is public, and is maintained by a developer rather
than a sysadmin. The homelab must be treated as production: changes are
reviewed, reproducible and reversible. A Windows control node is possible later
through WSL, so automation must not depend on POSIX shell scripts beyond a task
script that supplies default arguments.

## Decision

**Ansible.** Desired state, idempotent playbooks and no agent on the nodes. It
is the smallest tool that covers inventory, secrets handling and check mode, and
its YAML is reviewable in a public repository.

**mise as the single interface.** Every command is a mise task with a
description, so `mise tasks` documents the project and CI runs the same entry
points as a developer. Tasks are named after their subject and action, never
after the tool that implements them, so a name survives replacing the tool.
Toolchain versions live in `mise.toml`; `pin` and `lockfile` are enabled so
additions are exact and `mise.lock` records checksums. A `postinstall` hook
syncs the Python environment and installs collections and git hooks, so
`mise install` is the only setup step.

**Python environment through uv.** `pyproject.toml` declares the Python packages
and `uv.lock` pins them with their transitive dependencies, so the environment
is reproducible across machines and CI. The interpreter version lives in
`.python-version`, which uv reads and mise installs, and `requires-python` keeps
it in the series that both Ansible tools support. mise activates the environment
uv owns; `mise.toml` keeps no Python package of its own.

**One distribution channel per tool.** A Node project can delegate all its
tooling to pnpm, but this one cannot: PyPI does not distribute Biome or
Cocogitto and uv has no task runner. The split follows one test: a tool that
must see the project's Python packages, either by importing them or by sharing a
version with one that does, belongs to uv; every other tool belongs to mise,
even when PyPI distributes it. So mise installs the interpreter, the package
manager and every executable the quality gate runs, and uv installs the
environment the project executes in. `ansible-lint` and `yamllint` are the
justified exceptions: `ansible-lint` imports `ansible-core` and requires
`yamllint` as a library, so separating them would duplicate `ansible-core`, let
the versions drift and leave the transitive set outside a lock.

**lefthook for hooks.** A single binary, configured in `lefthook.yml`, validates
the commit message and runs the same lint tasks as CI, narrowed to the staged
Markdown, JSON and YAML files, while Ansible checks the whole subject because it
needs cross-file context. Hooks are the local, incremental gate; CI is the
shared, complete one. lefthook is installed outside the Python environment, so a
broken environment cannot disable the gate.

**SSH-alias inventory.** Hosts are named by SSH alias; `~/.ssh/config` owns
`HostName`, `User` and the key. The inventory never sets `ansible_host` or
`ansible_user`, which keeps identifying data out of a public repository and
gives each fact one owner. See `docs/control-node.md`.

**VPN-only access.** Nodes are not exposed publicly; the VPN is the only path to
them. Access-control changes are checked in check mode first and require
explicit approval, because a mistake can lock the maintainer out.

**ADRs for decisions.** Short ADRs next to the code explain context, decision,
alternatives and consequences, and a later ADR supersedes an older one instead
of rewriting it. A decision log lives with the code it explains and does not age
like a plan.

**Squash merge with Conventional Commits.** `main` stays linear and protected.
The PR title becomes the commit on `main`, so it follows the same Conventional
Commits format as local commits.

**Portable automation through pinned binaries.** Hooks are mise tasks that call
pinned binaries instead of repository-owned scripts: Biome formats and checks
JSON wherever it lives, Cocogitto validates commit messages, and only the
Ansible toolchain needs a Python environment. Task scripts never implement
logic, so hooks also work from Git Bash or WSL on Windows. Ansible itself still
requires a POSIX control node, hence WSL.

## Alternatives discarded

- **Shell plus Makefile:** fast to start, but no tool version management and no
  portable hooks; every command would be duplicated in CI.
- **asdf or plain pip and venv:** works, but versions and Python environments
  would be managed twice. mise already wraps both and provides tasks.
- **mise's PyPI backend for the Ansible toolchain:** one virtualenv per package,
  a second `ansible-core` that can drift from the one ansible-lint imports,
  transitive dependencies outside any lock and a `PATH` rewritten at run time.
- **All tooling in uv:** not possible, because Biome and Cocogitto are not on
  PyPI and uv has no task runner.
- **Unofficial PyPI wrappers for Biome:** a public repository that treats the
  homelab as production takes each binary from its project's own channel.
- **Download scripts for Biome and Cocogitto:** reimplement what mise already
  does and add repository code to test and maintain.
- **lefthook inside the Python environment:** ties the quality gate to the
  environment it validates.
- **mise `[deps.uv]`:** syncs the environment automatically, but it is
  experimental; an explicit task dependency does the same with stable behavior.
- **pre-commit:** Python-centric and hook definitions live in YAML per tool;
  lefthook runs any command directly and is a single binary.
- **Inventory with `ansible_host` and `ansible_user`:** self-contained, but
  publishes addresses and usernames and duplicates `~/.ssh/config`.
- **Exposing nodes through the internet:** larger attack surface, and access
  control would become part of this repository.
- **Roadmap document:** no durable record of why a decision was made, and stale
  by design.
- **Merge commits:** noisier history and a PR title that does not describe what
  landed.
- **Custom hook scripts:** a small script per check works, but duplicates what
  Biome and Cocogitto already do and becomes code to test and maintain. Both
  tools run from mise without adding a Node runtime.

## Consequences

- Adding a Python package means `uv add` and a new `uv.lock`; adding any other
  tool means editing `mise.toml` and refreshing `mise.lock`.
- Every new command must be a mise task, including the ones CI uses.
- A task that runs project code depends on the environment sync, which fails
  when `pyproject.toml` changed without a new `uv.lock`.
- The local gate checks the files a commit touches; CI checks the whole
  repository.
- The control node needs `~/.ssh/config` entries and the VPN; a fresh clone
  cannot reach nodes without them.
- `nodes:ping` is the only task that contacts a node, and it runs outside CI
  because it needs the VPN and `~/.ssh/config`; `site.yml` stays inert until a
  role arrives.
- The default branch only accepts squash-merged pull requests whose required
  checks pass; the ruleset in `.github/rulesets/` records that intent, and
  GitHub applies it only after it is uploaded through the API.
- Hook installation is part of the `mise install` postinstall hook, so a clone
  is fully set up once tools are installed and the environment is synced.
- On Windows, Ansible runs inside WSL even though hooks and tasks are portable.
