# 0006: Node access

## Status

Accepted.

## Context

Ansible reaches nodes over SSH, and this repository is public: whatever the
inventory says about a node is published with it.

## Decision

Hosts are named by SSH alias; `~/.ssh/config` owns `HostName`, `User` and the
key. The inventory never sets `ansible_host` or `ansible_user`, which keeps
identifying data out of a public repository and gives each fact one owner. See
`docs/control-node.md`.

Nodes are not exposed publicly; the VPN is the only path to them. Access-control
changes are checked in check mode first and require explicit approval, because a
mistake can lock the maintainer out.

## Reasons

A single owner for the address, the user and the key avoids the inventory and
`~/.ssh/config` disagreeing, and keeps values that identify the network out of a
public repository.

## Alternatives discarded

- **Inventory with `ansible_host` and `ansible_user`:** self-contained, but
  publishes addresses and usernames and duplicates `~/.ssh/config`.
- **Exposing nodes through the internet:** larger attack surface, and access
  control would become part of this repository.

## Consequences

- The control node needs `~/.ssh/config` entries and the VPN; a fresh clone
  cannot reach nodes without them.
