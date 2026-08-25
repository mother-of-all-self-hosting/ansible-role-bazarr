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

# Setting up Bazarr

This is an [Ansible](https://www.ansible.com/) role which installs [Bazarr](https://www.bazarr.media/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Bazarr is a companion application to [Sonarr](https://sonarr.tv/) and [Radarr](https://radarr.video/) that manages and downloads subtitles based on your requirements.

See the project's [documentation](https://wiki.bazarr.media/) to learn what Bazarr does and why it might be useful to you.

## Adjusting the playbook configuration

To enable Bazarr with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# bazarr                                                               #
#                                                                      #
########################################################################

bazarr_enabled: true

########################################################################
#                                                                      #
# /bazarr                                                              #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Bazarr you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
bazarr_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Configuring HTTP Basic authentication

Since there does not exist an authentication system on the web interface, this role is configured to enable the HTTP Basic authentication on Traefik by default, considering the nature of the service. See [this page](https://doc.traefik.io/traefik/reference/routing-configuration/http/middlewares/basicauth/) on the Traefik's documentation for details.

You can use `htpasswd` to generate the user and password pair, which needs to be set to `bazarr_container_labels_traefik_middleware_basic_auth_users`.

If another authentication service is used or authentication is not required at all, you can disable it by adding the following configuration to your `vars.yml` file:

```yaml
bazarr_container_labels_traefik_middleware_basic_auth_enabled: false
```

Bazarr's REST API is a separate matter: it is guarded by an API key that Bazarr generates for itself on first boot and keeps in `config/config.yaml` under `bazarr_data_path`. That guard applies regardless of whether the HTTP Basic authentication above is enabled.

### About the version that gets deployed

This role deploys the [LinuxServer.io Bazarr image](https://docs.linuxserver.io/images/docker-bazarr), pinned by `bazarr_version`.

Two things about that tag are worth knowing:

- **It is not immutable.** LinuxServer.io rebuilds its images regularly — for base image and dependency updates, without any change to Bazarr itself — and republishes `linuxserver/bazarr:<version>` each time. `linuxserver/bazarr:1.6.0` resolved to `v1.6.0-ls357`, then `-ls358`, `-ls359` and `-ls360` over the course of a single month. Since the role pulls the image on every installation run, you receive those rebuilds without the pin ever changing.
- **An immutable form exists.** LinuxServer.io also publishes `v<version>-ls<build>` tags, which do identify a single build. If you would rather pin one, set it yourself:

  ```yaml
  bazarr_version: v1.6.0-ls360
  ```

  Note that the role's own dependency updates track the `<version>` form, so a pin like this becomes yours to maintain from then on.

### About the port

`bazarr_container_http_port` is deliberately not adjustable. Bazarr reads its port from the configuration file it maintains for itself under `bazarr_data_path`, which this role does not template, and the container image's readiness check is hardwired to 6767 in any case — so the role refuses any other value instead of bringing up a service nothing can reach.

To change the port Bazarr is published on outside the container, use `bazarr_container_http_host_bind_port`.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `bazarr_environment_variables_additional_variables` variable

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Bazarr becomes available at the specified hostname like `https://example.com`. Open the URL in a browser to reach its web interface — Bazarr has no accounts of its own, so what guards it is the HTTP Basic authentication described above.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu bazarr` (or how you/your playbook named the service, e.g. `mash-bazarr`).
