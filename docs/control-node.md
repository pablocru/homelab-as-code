# Control node

The control node is the machine where `mise run` executes. It holds everything
that is specific to this homelab and must not be published. Ansible reads the
connection details there, never from this repository.

## Prerequisites

Control node:

- mise installed.
- An `~/.ssh/config` entry as described below.
- The VPN client joined.

Node:

- Ubuntu Server 24.04 LTS.
- VPN and firewall already set up.
- SSH access with the control node's key.
- A login user that can escalate with sudo; the password is prompted at run
  time.
- `/usr/bin/python3` present.

## SSH configuration

Host aliases resolve through the standard OpenSSH client configuration at
`~/.ssh/config`. The inventory names only the alias (`homelab-node01`), so
Ansible delegates the address, the user and the key to OpenSSH. That keeps
identifying data out of a public repository and lets the same inventory work on
any control node.

Expected format, with placeholders instead of real values:

```sshconfig
Host <alias>
  HostName <address>
  User <username>
  IdentityFile ~/.ssh/<private_key>
  IdentitiesOnly yes
```

`Host` is the alias used in `inventories/production/hosts.yml`, `HostName` is
the address the alias resolves to, `User` is the login account and
`IdentityFile` is the private key. `IdentitiesOnly yes` stops the client from
offering unrelated keys.

## Why the inventory has no `ansible_host` or `ansible_user`

Both variables would duplicate what `~/.ssh/config` already owns. A single
source avoids the two disagreeing and keeps connection details out of the
repository. `ansible.cfg` therefore sets only the inventory path, the local
collections path, the roles path and the remote Python interpreter.

## Other values outside the repository

No secrets or private values exist yet. When one is needed, a new ADR chooses
the mechanism and this document records its expected format and placeholders.

## Access

Nodes are reachable only through the VPN. That configuration belongs to the
control node and is out of scope for this document: playbooks only assume that
the alias resolves and the SSH connection works. The node is prepared by hand
before this repository takes over, see `docs/adr/0007-manual-node-bootstrap.md`.
