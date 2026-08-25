<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024, 2026 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up FlareSolverr

This is an [Ansible](https://www.ansible.com/) role which installs [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

FlareSolverr is an open-source proxy server to bypass Cloudflare protection.

See the project's [documentation](https://github.com/FlareSolverr/FlareSolverr/blob/master/README.md) to learn what FlareSolverr does and why it might be useful to you.

## Adjusting the playbook configuration

To enable FlareSolverr with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# flaresolverr                                                         #
#                                                                      #
########################################################################

flaresolverr_enabled: true

########################################################################
#                                                                      #
# /flaresolverr                                                        #
#                                                                      #
########################################################################
```

### Exposing the instance (optional)

By default, the FlareSolverr instance is not exposed externally, as it is mainly intended to be used in the internal network, connected to other services.

To expose it to the internet, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
flaresolverr_hostname: "example.com"

flaresolverr_container_labels_traefik_enabled: true
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `flaresolverr_environment_variables_additional_variables` variable

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

FlareSolverr has no web interface and no accounts. It is a JSON API that other services call, and it offers exactly three routes:

- `GET /` — reports the version and the browser's user agent
- `GET /health` — always answers `{"status": "ok"}`, without checking anything
- `POST /v1` — the only route that does anything, taking a `cmd` of `sessions.create`, `sessions.list`, `sessions.destroy`, `request.get` or `request.post`

Point the service that needs it (Prowlarr, Jackett, and similar) at the instance. Within the same container network that is `http://flaresolverr:8191`; from elsewhere it is whatever `flaresolverr_container_http_host_bind_port` publishes, or the hostname you exposed it at.

To check by hand that it can really solve a page:

```sh
curl -s -X POST -H 'Content-Type: application/json' \
  -d '{"cmd": "request.get", "url": "https://example.com", "maxTimeout": 60000}' \
  http://127.0.0.1:8191/v1
```

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu flaresolverr` (or how you/your playbook named the service, e.g. `mash-flaresolverr`).

### Do not trust `/health`

`/health` is a hard-coded `{"status": "ok"}` that never touches the browser, and `GET /` reports a user agent FlareSolverr cached while starting up. Neither notices a browser that has stopped working. A `request.get` like the one above is the only request that tells you anything.

Note also that `POST /v1` answers with HTTP 200 in cases where it did not do what was asked: it is the `status` field of the JSON body that says whether a request succeeded. Chromium's own error pages come back as a perfectly successful `"status": "ok"` response, so a check worth having reads the returned content too.
