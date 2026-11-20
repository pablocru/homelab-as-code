# 0012: Patching posture

## Status

Accepted.

## Context

The `base` role enables daily unattended security upgrades and promises that a
node never reboots on its own. Until now it wrote only the two keys it found in
`20auto-upgrades`, so the rest of the patching behaviour was whatever apt
defaults happened to be.

A read of the running configuration on a managed node shows what that leaves in
the air: `APT::Periodic::AutocleanInterval` is `0`, so the downloaded packages
are never removed, and no automatic-reboot key exists anywhere on the node, so
the promise depends on defaults that this repository neither states nor reviews.

Two different keys can reboot a node. `APT::Periodic::Automatic-Reboot` is read
by apt's periodic mechanism and `Unattended-Upgrade::Automatic-Reboot` by
unattended-upgrades, which parses the merged apt configuration itself. They are
independent: declaring one says nothing about the other.

## Decision

`20auto-upgrades` declares the whole posture, not only the two keys that were
already there:

- `APT::Periodic::Automatic-Reboot "0"` and
  `Unattended-Upgrade::Automatic-Reboot "false"`, one per mechanism.
- `APT::Periodic::AutocleanInterval "7"`, so archives older than a week go.

## Reasons

A default is a value without an owner. It changes between releases, and nothing
in the repository would show that the node started behaving differently. Stating
a value that already is the default costs one line and turns an accident into
state a reviewer can see and a diff can contradict.

The two reboot keys are the case that makes this necessary rather than tidy: apt
and unattended-upgrades are separate programs, each with its own switch, so a
single line would look like the whole posture while leaving the other path open.
The cache has to shrink for the disk to stay predictable: one week is long
enough to reinstall a package without the network and short enough that the node
does not accumulate months of archives.

## Alternatives discarded

- **Rely on the distribution defaults:** what the repository did until now, and
  the reason the posture had to be measured instead of read.
- **A file of its own, numbered after the package's:** apt reads `apt.conf.d` in
  filename order, so a `99-*` file would win, but the posture would end up split
  across two files: the two daily keys in the package's file and the rest in
  ours, with which one wins decided by a number a reader has to know.
- **Declare only the reboot keys:** leaves the cache growing without limit, the
  other default that changes silently.
- **Declare `AutocleanInterval "0"`:** states "never clean" as a decision, which
  is the opposite of what the node should do.
- **`Automatic-Reboot-Time`:** only read when a reboot is enabled, and both keys
  say off, so it would be a line that cannot take effect.
- **Clean the cache from a task or a handler of its own:** a second scheduler
  owning what apt already schedules.

## Consequences

- How often the cache is cleaned is a value in this file, not a new task.
- No package in the dpkg database owns `20auto-upgrades`, so dpkg never treats
  it as a conffile: `unattended-upgrades` ships a template and `ucf` writes the
  file.
- `ucf` already records the file as changed, so a package upgrade keeps the
  local copy instead of replacing it, and may leave a `*.ucf-*` file beside it,
  which apt ignores.
- The role writes the file whole, so a key added by hand disappears on the next
  apply.
- An installed kernel still takes effect only after a manual reboot. Knowing
  when one is pending is a separate decision, not part of this one.
