<!--
SPDX-FileCopyrightText: 2025 sudo-Tiz

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Flaresolverr Ansible role

This is an [Ansible](https://www.ansible.com/) role which installs [Flaresolverr](https://flaresolverr.dev/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

This role _implicitly_ depends on:

- [`com.devture.ansible.role.playbook_help`](https://github.com/devture/com.devture.ansible.role.playbook_help)
- [`com.devture.ansible.role.systemd_docker_base`](https://github.com/devture/com.devture.ansible.role.systemd_docker_base)

Check [defaults/main.yml](defaults/main.yml) for the full list of supported options.

For an Ansible playbook which integrates this role and makes it easier to use, see the [mash-playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

## Limitations

This role configures **Flaresolverr** securely by:

1. Running the container as a non-root user.
2. Making the filesystem read-only.
3. Dropping all capabilities.

Due to Dockerfile limitations, the container must run as the **flaresolverr** user (`uid:1000`, `gid:1000`). You can change these values with:

- `flaresolverr_docker_uid`
- `flaresolverr_docker_gid`
