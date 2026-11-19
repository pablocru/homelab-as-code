# 0003: Target OS

## Status

Accepted.

## Context

The node runs Ubuntu Server 22.04 LTS, and playbooks, module behaviour and the
remote Python interpreter depend on which distribution and release it is. A
second distribution would make the homelab harder to reason about.

## Decision

The target OS is Ubuntu Server 22.04 LTS, the release the node runs and the one
every check validates. `ansible.cfg` pins `interpreter_python` to
`/usr/bin/python3`, the path the distribution keeps as its default interpreter.

## Reasons

Declaring what runs keeps the repository honest: `site:check` and `site:apply`
exercise 22.04, so 22.04 is what the gate can vouch for. Pinning the interpreter
by path keeps module execution reproducible across reboots and package updates,
instead of depending on whichever Python discovery finds first.

## Alternatives discarded

- **Declare 24.04 before the node runs it:** the repository would describe a
  platform no check has touched. The upgrade arrives as its own ADR, which
  supersedes this one.
- **Debian:** equally supported by Ansible, but the node already runs Ubuntu, so
  moving would be a second manual step with no benefit.
- **Interpreter auto-discovery:** picks another interpreter as soon as one
  appears on the node, so the same playbook would behave differently over time.

## Consequences

- 22.04 reaches the end of standard support in 2027, so the release upgrade to
  the next LTS will happen and will need its own ADR.
- That ADR supersedes this one, because the target platform changes rather than
  the reasons behind choosing one.
- Roles written and validated here must be re-checked after the upgrade: the
  path `/usr/bin/python3` survives, the version behind it does not.
