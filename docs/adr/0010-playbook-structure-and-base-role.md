# 0010: Playbook structure and base role

## Status

Accepted.

## Context

The repository has an inventory and a way to reach the node, but no playbook.
Every future capability needs an entry point, a convention for packaging it and
a way to gain root on the node.

## Decision

`playbooks/site.yml` is the single entry point and applies roles to the
`homelab` group. Each role lives in `roles/` and has one responsibility. The
first role, `base`, is the baseline of every node: it asserts the target OS,
installs a short list of packages, sets the time zone (UTC by default) and
enables unattended security upgrades without automatic reboots.

The play escalates with `become`. The sudo password is prompted at run time with
`--ask-become-pass` in the `site:*` tasks and is never stored.

## Reasons

One entry point makes the order of roles explicit and reviewable. Roles keep
each responsibility testable and replaceable, and `base` is small enough to be
the first change that exercises the whole circuit: check mode, apply and a
second apply that reports no changes. Prompting for the password keeps root
access dependent on something that is not on disk. Automatic reboots are off
because an unexpected reboot of a node reachable only through the VPN is a risk
the maintainer should choose. The time zone defaults to UTC because it has no
daylight saving changes, so scheduled jobs and log timestamps stay unambiguous.

## Alternatives discarded

- **One playbook per concern:** several entry points, with an execution order
  that lives in the maintainer's head.
- **Tasks written directly in the playbook:** not reusable and unreadable as it
  grows.
- **Passwordless sudo:** anyone who obtains the SSH key becomes root.
- **Storing the sudo password encrypted:** needs a secrets mechanism that has
  not been decided yet.

## Consequences

- Every new capability is a role added to `playbooks/site.yml`.
- `site:check` and `site:apply` ask for the sudo password and cannot run
  unattended, so only the maintainer runs them.
- Kernel updates installed automatically take effect only after a manual reboot.
- `community.general` is a pinned dependency, so `ansible:lint` and the `site:*`
  tasks install collections first.
- Changing the time zone is a change to `base_timezone`, not to the role.
