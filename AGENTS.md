# AGENTS.md

Permanent instructions for coding agents in this repository. Keep it accurate:
if you change tooling or conventions, update this file in the same PR. State the
outcome a rule protects, not the technique. Do not duplicate facts that have a
source of truth elsewhere (commands, versions, hooks).

## Project

`homelab-as-code`: Ansible-based infrastructure as code for a personal homelab.
Public repository. The maintainer is a developer, not a sysadmin: every choice
must be explained, documented and justified, not necessarily simple. Treat the
homelab as production.

**Everything is in English**: code, comments, variable names, docs, commit
messages, PR text. Only the chat with the maintainer may be in Spanish.

## Where each rule lives

- `AGENTS.md`: invariants, policy, boundaries.
- `mise.toml`: toolchain versions and every command. Source of truth for
  commands.
- `pyproject.toml` and `uv.lock`: Python packages and the environment they
  build; `.python-version` states the interpreter version.
- `lefthook.yml`: checks that must pass before a commit. Source of truth for
  what "done" means locally.
- `docs/adr/`: decisions and their reasons.

## Decisions (ADRs)

- A change that introduces or reverses a decision (tool, structure, security
  posture, a new external dependency such as a private repo) includes an ADR in
  the same PR. Routine changes do not need one.
- ADRs are short and numbered: context, decision, reasons, alternatives
  discarded, consequences. Do not rewrite history in an ADR: to change a
  decision, add a new ADR and set the old one's status to "Superseded by NNNN".
- One decision per ADR; one PR may contain several ADRs. To change one decision
  inside an ADR that records several, add a new ADR and mark the old one
  "Superseded in part by NNNN: section name" instead of rewriting it.
- Prefer a code comment or a doc line over an ADR when the reason is local.

## Invariants

- Everything is reproducible from this repo. A manual change on a node is
  codified in the same PR or recorded as an exception in an ADR.
- Playbooks are idempotent: a second run reports no changes.
- **A broken environment fails the gate; it never skips it.** Hooks and CI run
  the same mise tasks, and a missing or broken Python environment makes them
  fail instead of being skipped.
- **Each dependency has one owner.** Python packages and the interpreter version
  belong to the Python environment; every other tool belongs to the toolchain
  manager. Nothing is installed twice.
- Nodes are reachable only through the VPN. Access-control changes (SSH,
  firewall, VPN) are verified in check mode first and need explicit developer
  approval. Never apply rules that could lock the maintainer out.
- **The repo is public: never commit secrets or identifying data** (keys,
  tokens, emails, public IPs, network names, LAN ranges).
- Be transparent about what lives outside the repo: every detail a reader would
  expect to find here and does not (connection details, secrets, private values)
  is documented in `docs/` with its expected format and placeholders, and the
  reason it is not here. Do not document VPN-specific configuration beyond what
  a role needs to be understood.
- Connection details (`HostName`, `User`, key) live in the control node's
  `~/.ssh/config`; the inventory uses the host alias only and never sets
  `ansible_host` or `ansible_user`.

## Commands

- Run every command through `mise`. `mise.toml` is the only place where commands
  are defined; do not call underlying tools directly when a task exists.
- **Any command that writes (local files outside the working tree, remote nodes,
  remote GitHub objects) needs explicit approval, and any command not defined in
  `mise.toml` too.** If you need a command that is not supported, justify it,
  and after approval add it to `mise.toml` with a description.
- Read-only inspection (listing, viewing, linting, syntax and check mode) may
  run without asking.
- **Never change a remote node.** Prepare the change and hand it over.
- A task that needs the Python environment declares it as a dependency, so a
  fresh clone works from `mise install` without a manual sync step.
- Tool versions are pinned exactly, including the ones `mise` installs. Never
  use floating versions.

## Code conventions

- Layout: `inventories/`, `playbooks/`, `roles/`, `collections/`, `docs/`. One
  responsibility per role.
- Fully qualified collection names; every task has a name; role variables are
  prefixed with the role name; defaults in `defaults/main.yml`.
- Task names state the subject and the action (`yaml:lint`), never the tool that
  implements it, so a name survives replacing the tool.
- `command`/`shell` only when no module exists, with `changed_when` and
  check-mode handling. Restarts go through handlers.
- Pin collections and any package whose version matters. No `latest`.
- KISS: the smallest change that meets the goal, no options nothing requires.
  DRY facts, not shapes.
- A comment explains a decision the code cannot show: why a setting exists and
  what breaks if it is removed. Do not restate what the line already says.
- Verify module parameters in the installed version's docs or source code; do
  not rely on memory.
- When adding or changing a rule, update stale or duplicated references in the
  same change.

## Git

- **`main` is protected and read-only. Never commit there; warn the developer.**
- **No force push. No rewriting pushed history.**
- Ask before commands that discard local work.
- Every change goes on a new branch from an up-to-date `main`, named with a
  Conventional Commits type (`feat/…`, `fix/…`, `chore/…`, `docs/…`).
- Conventional Commits: `type(scope): imperative summary`, lowercase, no period,
  max 72 chars. Small, focused commits. Hooks verify only the format: the rest
  is a convention.
- Git hooks are managed with lefthook. Warn the developer if they are not
  installed. Never bypass them (`--no-verify`).
- **Never push or open a PR unless asked.**

## GitHub

- Prefer `gh`. Read-only commands are allowed; anything that creates or edits a
  remote object needs approval (show the exact command first).
- The PR title is the commit that lands on `main` (squash merge): write it as a
  Conventional Commit. Follow `.github/pull_request_template.md`.
- The developer merges and closes. Never delete anything on GitHub. Never run
  `gh auth` or token commands.

## Done means

- The checks in `lefthook.yml` pass.
- If the change touches a node's configuration, check mode against the node also
  passes (see `mise.toml` for the task).
- The final report states what changed, which checks ran, and what could not be
  verified. Never report a check as passed if it was not run.

## Working method

- Work only with the files the task needs; do not read the whole repo.
- If the task conflicts with this file or is ambiguous, stop and ask.
