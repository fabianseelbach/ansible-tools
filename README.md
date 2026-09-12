# Ansible Tools

A Collection (not ansible one) of tools I wrote in Ansible to make my life easier.

> [!warning]
> I did not add Vars for like Ports and Paths and Names, as I do not need them.

## Install

git clone https://github.com/fabianseelbach/ansible-tools
cd ansible-tools
ansible-galaxy collection install -r requirements.yml

## Playbooks

### GitLab Upgrade

`gitlab-upgrade.yml` upgrades an existing GitLab installation running as a
Podman container. The playbook pulls the requested GitLab image, stops and
replaces the existing container, starts the associated systemd unit, and then
waits for the background migrations to complete. Any unused GitLab images are
removed afterward.

Before running the playbook, check the GitLab upgrade path here:
https://gitlab-com.gitlab.io/support/toolbox/upgrade-path/

The target host must be included in the `gitlab_host` inventory group. Before
running the playbook, install the collection dependencies and provide the
GitLab version:

```bash
ansible-playbook -i inventory.ini gitlab-upgrade.yml \
	-e gitlab_version=17.11.0
```

#### Variables

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `gitlab_version` | Yes | - | GitLab version to use, for example `17.11.0`. |
| `gitlab_edition` | No | `ce` | GitLab edition used in the image name, for example `ce` or `ee`. |
| `gitlab_user` | No | `gitlab` | User under which Podman container management and the user systemd unit run. |

### Nexus Update

`nexus-update.yml` upgrades an existing Nexus Repository Manager installation
running as a Podman container. The playbook pulls the requested Nexus image,
stops the existing container, and recreates it with persistent Nexus data at
`/var/lib/containers/nexus/data/nexus`. After restarting, it checks whether the
Nexus status API is reachable on port `8081`. Any unused Nexus images are then
removed.

The playbook runs on `nexus_host` and requires the collection dependencies from
`requirements.yml` and a specified Nexus version:

```bash
ansible-playbook -i inventory.ini nexus-update.yml \
	-e nexus_version=3.75.1
```

#### Variables

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `nexus_version` | Yes | - | Nexus version for the Sonatype image, for example `3.75.1`. |
| `nexus_user` | No | `nexus` | User under which Podman container management and the user systemd unit run. |

