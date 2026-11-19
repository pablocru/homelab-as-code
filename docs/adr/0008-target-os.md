# 0008: Target OS

## Status

Accepted.

## Context

Playbooks, module behaviour and the remote Python interpreter depend on which
distribution and release a node runs, and a second distribution would make the
homelab harder to reason about.

## Decision

The target OS is Ubuntu Server 24.04 LTS, reached by a release upgrade from
22.04 done by hand (see 0007). `ansible.cfg` pins `interpreter_python` to
`/usr/bin/python3`, the path the distribution keeps as its default interpreter.

## Reasons

24.04 is an LTS release, so it receives security updates for years without a
release upgrade. Pinning the interpreter by path keeps module execution
reproducible across reboots and package updates, instead of depending on
whichever Python discovery finds first. `nodes:ping` succeeded against the node
on 24.04, which proves that interpreter runs there.

## Alternatives discarded

- **Debian:** equally supported by Ansible, but the node already runs Ubuntu, so
  moving would be a second manual step with no benefit.
- **Interpreter auto-discovery:** picks another interpreter as soon as one
  appears on the node, so the same playbook would behave differently over time.

## Consequences

- The pin survives a release upgrade because it names a path; the version behind
  the path changes.
