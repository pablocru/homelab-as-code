# 0007: Manual node bootstrap

## Status

Accepted.

## Context

The node was prepared by hand once, before any code existed: OS install and
release upgrade, VPN and firewall set up following the vendor documentation, and
the SSH key created from the control node.

## Decision

The repository assumes a prepared node: a reachable SSH alias and the VPN.

## Reasons

Ansible needs SSH and the VPN to reach a node, so it cannot create them. It is a
one-off step, and its values identify the network, so they must stay out of a
public repository.

## Alternatives discarded

- **Codify the bootstrap in this repository:** it would have to run before the
  repository can reach the node.

## Consequences

- This repository does not verify the firewall and VPN state.
- Later manual changes still follow the invariant in `AGENTS.md`.
- The prerequisites are listed in `docs/control-node.md`.
