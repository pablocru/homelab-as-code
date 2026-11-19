# 0003: Target OS

## Status

Accepted.

## Context

The node runs a Linux distribution, and playbooks, module behaviour and the
remote Python interpreter depend on which one it is. A second distribution would
make the homelab harder to reason about.

## Decision

The target OS is Ubuntu Server 24.04 LTS. The node was upgraded from 22.04 by
hand before any code existed; that one-off manual step is the recorded exception
to "everything is reproducible from this repo". `ansible.cfg` pins
`interpreter_python` to `/usr/bin/python3`, the interpreter the distribution
ships.

## Reasons

24.04 is the current LTS, so it receives security updates for years without a
release upgrade. Pinning the distribution interpreter keeps module execution
reproducible across reboots and package updates, instead of depending on
whichever Python discovery finds first.

## Alternatives discarded

- **Debian:** equally supported by Ansible, but the node already ran Ubuntu, so
  moving would be a second manual step with no benefit.
- **Interpreter auto-discovery:** picks another interpreter as soon as one
  appears on the node, so the same playbook would behave differently over time.

## Consequences

- The pin is the distribution default; a release upgrade that moves it makes the
  pin stale.
- The manual upgrade from 22.04 is not reproducible from the repository.
