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

Tests a standard Bazarr installation.

Because Bazarr's systemd unit is `Restart=always`, a crash-looping container still reports `active`, so the scenario does not settle for a running unit. What it establishes instead:

- **The pinned image is what runs.** The container's image reference has to be `linuxserver/bazarr:<bazarr_version>`, and the image's `org.opencontainers.image.version` label has to read `v<bazarr_version>-ls<build>`. That second half matters: `linuxserver/bazarr:<version>` is a tag LinuxServer.io *republishes* on every rebuild, so on its own it does not identify a build — the label does, and asserting the pair records which Bazarr release the tag currently resolves to.
- **The role's configuration reaches the process, not merely the disk.** The scenario asks for a timezone (`Australia/Eucla`) that is neither the role's default nor anything the image would arrive at by itself, plus a marker in `bazarr_environment_variables_additional_variables`. Both are checked in the rendered `env` file and in the running container's environment, and the timezone is additionally checked against what Bazarr itself reports back over its API.
- **The API is guarded.** Bazarr ships no authentication on its web interface, but its REST API requires the key it generates for itself on first boot. The scenario asserts that an unauthenticated request is refused, that the key found in the configuration file on the role's data path opens it, and that a different key does not.
- **State lands on the role's data path.** The `/config` bind mount has to be `bazarr_data_path`, Bazarr's own configuration file has to be there, and the SQLite database there has to carry a migrated Bazarr schema with an Alembic revision stamped. The service is then restarted and has to come back answering to the key it persisted, which is what proves the read-only container keeps nothing of value outside the mount.
- **Bazarr listens where the role publishes.** `bazarr_container_http_port` is asserted against the port in Bazarr's own configuration file, so the two cannot drift apart unnoticed.

These checks are what an automated version bump would have to survive. A Bazarr container that is hardened and versioned identically but was not deployed by this role is rejected at the data-path assertion.

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
