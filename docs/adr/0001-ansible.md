# 0001: Ansible

## Status

Accepted.

## Context

`homelab-as-code` starts from an empty repository and manages a personal homelab
from a Linux control node. It is public and maintained by a developer rather
than a sysadmin, and the homelab must be treated as production: changes are
reviewed, reproducible and reversible.

## Decision

Ansible. Desired state, idempotent playbooks and no agent on the nodes.

## Reasons

It is the smallest tool that covers inventory, secrets handling and check mode,
and its YAML is reviewable in a public repository.

## Alternatives discarded

None recorded.

## Consequences

None recorded.
