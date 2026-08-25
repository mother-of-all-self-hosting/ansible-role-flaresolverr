<!--
SPDX-FileCopyrightText: 2023, 2026 Slavi Pantaleev
SPDX-FileCopyrightText: 2025 sudo-Tiz
SPDX-FileCopyrightText: 2025, 2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# FlareSolverr Ansible role

This is an [Ansible](https://www.ansible.com/) role which installs [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

This role *implicitly* depends on:

- [`com.devture.ansible.role.playbook_help`](https://github.com/devture/com.devture.ansible.role.playbook_help)
- [`com.devture.ansible.role.systemd_docker_base`](https://github.com/devture/com.devture.ansible.role.systemd_docker_base)

Check [`defaults/main.yml`](defaults/main.yml) for the full list of supported options. Refer to [this page](docs/configuring-flaresolverr.md) for details about setting up the service with this role.

💡 For an Ansible playbook which integrates this role and makes it easier to use, see the [Mother-of-All-Self-Hosting Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

## Limitations

This role configures **FlareSolverr** securely by:

1. Running the container as a non-root user.
2. Dropping all capabilities.

Due to Dockerfile limitations, the container must run as the **flaresolverr** user (`uid:1000`, `gid:1000`). You can change these values with:

- `flaresolverr_docker_uid`
- `flaresolverr_docker_gid`

## Development

### pre-commit

You can optionally install a Git pre-commit hook (via [mise](https://mise.jdx.dev/) + [prek](https://prek.j178.dev/)) that runs formatting and linting checks before each commit. See [`.pre-commit-config.yaml`](./.pre-commit-config.yaml) for which hooks are to be executed.

To install the hook, run the [`just`](https://github.com/casey/just) command below:

```sh
just prek-install-git-pre-commit-hook
```

### Molecule

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

Refer to [this page](./molecule/README.md) for details about how to utilize it.

### Releases

Release tags (`v<FlareSolverr version>-<release>`, e.g. `v3.5.0-2`) are cut automatically by the [autotag workflow](.github/workflows/autotag.yml) when a commit lands on `main`. Nobody has to tag anything by hand.

The tag is derived from the state of the repository — `flaresolverr_version` in [`defaults/main.yml`](defaults/main.yml), plus the tags that already exist — rather than from commit messages, which makes it independent of the order in which pull requests get merged. A commit that only touches documentation or CI does not get a release.

[`bin/compute-next-tag.sh`](bin/compute-next-tag.sh) holds all of that logic and can be run locally to see what the currently checked out commit would be released as. [`bin/test-compute-next-tag.sh`](bin/test-compute-next-tag.sh) exercises it against throwaway repositories, and runs as a prek hook whenever that script or `defaults/main.yml` changes.
