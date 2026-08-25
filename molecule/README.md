<!--
SPDX-FileCopyrightText: 2018-2026 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently there is one testing scenario available.

### `default`

Tests a standard FlareSolverr installation, and then makes it actually solve a page.

FlareSolverr is a headless Chromium behind a JSON API, and almost every surface it offers will answer happily while the browser is broken: `/health` is a hard-coded `{"status": "ok"}` that never touches the browser, `GET /` returns a user agent it cached during startup, and `request.get` reports `"status": "ok"` with `solution.status` 200 even for Chromium's own error pages. The scenario therefore starts a sidecar web server on the role's container network — running the very image the role deployed, so nothing extra is pulled — and asks FlareSolverr to fetch a page from it whose marker text is painted in by JavaScript over a placeholder saying the opposite. Getting that marker back means a real browser really fetched and really executed the page. The same request against a page the sidecar does not have, and against a port nothing listens on, establishes that not just any answer passes.

The scenario also asserts that the version FlareSolverr reports equals `flaresolverr_version` from `defaults/main.yml` (as does the running image's OCI version label), that the unit has not been restarting (`Restart=always` makes even a crash-looping container report `active`), that the container is unprivileged with all capabilities dropped, that a volume from `flaresolverr_container_additional_volumes_custom` really reached the container, and that browser sessions can be created, listed and destroyed. The port and the timezone it uses are deliberately not the defaults, so they can only have arrived through the `env` file this role renders.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
