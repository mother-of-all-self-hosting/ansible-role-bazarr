<!--
SPDX-FileCopyrightText: 2023, 2026 Slavi Pantaleev
SPDX-FileCopyrightText: 2025 spatterlight
SPDX-FileCopyrightText: 2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 sudo-Tiz

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Bazarr Ansible role

This is an [Ansible](https://www.ansible.com/) role which installs [Bazarr](https://www.bazarr.media/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

This role *implicitly* depends on:

- [`com.devture.ansible.role.playbook_help`](https://github.com/devture/com.devture.ansible.role.playbook_help)
- [`com.devture.ansible.role.systemd_docker_base`](https://github.com/devture/com.devture.ansible.role.systemd_docker_base)

Check [defaults/main.yml](defaults/main.yml) for the full list of supported options. Refer to [this page](docs/configuring-bazarr.md) for details about setting up the service with this role.

💡 For an Ansible playbook which integrates this role and makes it easier to use, see the [Mother-of-All-Self-Hosting Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

## Limitations

This role configures Bazarr with security in mind by doing the following:

1. Running the container as a non-root user
2. Making the filesystem read-only
3. Dropping most capabilities

Unfortunately, due to upstream requirements, some admissions had to be made:

1. Several capabilities related to permissions are added to the container
   - SETUID
   - SETGID
   - CHOWN
   - FOWNER
   - DAC_OVERRIDE
2. A `tmpfs` volume is mounted with `exec` permissions

You can read more about these upstream requirements in the documentation:

1. <https://docs.linuxserver.io/misc/non-root/>
2. <https://docs.linuxserver.io/misc/read-only/>

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

Release tags are cut automatically on every push to `main` by [`.github/workflows/autotag.yml`](.github/workflows/autotag.yml), which asks [`bin/compute-next-tag.sh`](bin/compute-next-tag.sh) what the commit should be released as.

The answer is derived from the repository's state — `bazarr_version` in [`defaults/main.yml`](defaults/main.yml) and the tags that already exist — rather than from commit messages, which makes it independent of the order in which pull requests get merged:

- a Bazarr version that has never been released starts a fresh counter (`v1.6.1-0`)
- any later change to `defaults/`, `meta/`, `tasks/` or `templates/` increments it (`v1.6.1-1`)
- a change that only touches documentation, CI configuration or the Molecule scenario is not released at all, since it does not change what a playbook run does

[`bin/test-compute-next-tag.sh`](bin/test-compute-next-tag.sh) exercises that logic against throwaway repositories, and runs as a pre-commit hook whenever it, the script it tests, or `defaults/main.yml` changes.
