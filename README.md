# homelab-as-code

My homelab, defined entirely as code with Ansible. I am a developer, not a
sysadmin, so this repository documents the journey: every change is code and
every decision is written down with its reason.

## Principles

- **Everything after the node bootstrap is reproducible.** Once a node is
  reachable over SSH and the VPN, every part of its configuration is versioned
  and can be rebuilt through code. The bootstrap itself is manual and documented
  in `docs/control-node.md` and `docs/adr/`.
- **Treat it as production.** Idempotent playbooks, a protected `main`, CI, and
  checks before changes reach a real node.
- **Decisions are documented.** The _why_ lives in `docs/adr/`.
- **Public by default, secrets never.** What the repo expects from outside (SSH
  config, private values) is documented with placeholders.
- **One interface.** `mise tasks` lists every command.

## Contribute

If you notice any mistakes or have suggestions, I’m all ears! I appreciate any
feedback, so don't hesitate to
[open an Issue on GitHub](https://github.com/pablocru/homelab-as-code/issues) or
submit a `Pull Request`.

### Contribution guidelines

1. `Fork` the repository and make your changes in a `new branch`.
1. Document your additions.
1. Use [Conventional Commits](https://www.conventionalcommits.org).
1. Submit a `Pull Request` following
   [the pull request template](.github/pull_request_template.md).

Thank you for helping improve this project!
