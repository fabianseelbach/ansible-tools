# Ansible Tools

A collection of tools I wrote in [Ansible](https://www.ansible.com/) to make my life easier.

Source code: [github.com/fabianseelbach/ansible-tools](https://github.com/fabianseelbach/ansible-tools)

> [!warning]
> I did not add Vars for like Ports and Paths and Names, as I do not need them.

> [!note]
> All playbooks are tested on [Rocky Linux 9](https://rockylinux.org/) and
> [CentOS 8](https://www.centos.org/). They are intended to work on [Red Hat Enterprise Linux](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)
> and other RHEL-derived distributions as well, but these are not tested.

## Install

```bash
git clone https://github.com/fabianseelbach/ansible-tools
cd ansible-tools
ansible-galaxy collection install -r requirements.yml
```

## Playbooks

### Fail2Ban Whitelist

[`fail2ban_whitelist.yml`](fail2ban_whitelist.yml) updates the `ignoreip` setting in
`/etc/fail2ban/jail.local` so that trusted infrastructure and host addresses are not blocked by
[Fail2Ban](https://www.fail2ban.org/wiki/index.php/Main_Page). The playbook fetches the
[Better Stack](https://betterstack.com/) IP list from its [status page IP list](https://uptime.betterstack.com/ips.txt),
collects the IPv4 and IPv6 addresses from all target hosts, merges them with the default local loopback
addresses, and writes the combined list into Fail2Ban's default section.

#### Run

```bash
ansible-playbook -i inventory.ini fail2ban_whitelist.yml
```

#### Variables

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `ignore_ip` | No | `['127.0.0.1/8', '::1/128']` | Base addresses that are always kept in the fail2ban ignore list. The playbook appends Better Stack and host IPs to this list. |

### GitLab Upgrade

[`gitlab-upgrade.yml`](gitlab-upgrade.yml) upgrades an existing
[GitLab](https://about.gitlab.com/) installation running as a
[Podman](https://podman.io/) container. The playbook pulls the requested GitLab image, stops and
replaces the existing container, starts the associated systemd unit, and then
waits for the background migrations to complete. Any unused GitLab images are
removed afterward.

Before running the playbook, check the [GitLab upgrade path](https://gitlab-com.gitlab.io/support/toolbox/upgrade-path/).

The target host must be included in the `gitlab_host` inventory group in
[`inventory.ini`](inventory.ini).

#### Run

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

[`nexus-update.yml`](nexus-update.yml) upgrades an existing
[Nexus Repository Manager](https://www.sonatype.com/products/sonatype-nexus-repository)
installation running as a [Podman](https://podman.io/) container. The playbook pulls the requested
Nexus image, stops the existing container, replaces the existing container and starts the associated
systemd unit. After restarting, it checks whether the Nexus status API is reachable on port `8081`.
Any unused Nexus images are then removed.

#### Run

```bash
ansible-playbook -i inventory.ini nexus-update.yml \
	-e nexus_version=3.75.1
```

#### Variables

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `nexus_version` | Yes | - | Nexus version for the Sonatype image, for example `3.75.1`. |
| `nexus_user` | No | `nexus` | User under which Podman container management and the user systemd unit run. |

### Roundcube Update

[`roundcube-update.yml`](roundcube-update.yml) updates an existing
[Roundcube](https://roundcube.net/) installation served by [Apache](https://httpd.apache.org/).
It first backs up the document root and database,
then downloads the requested release from [Roundcube's GitHub releases](https://github.com/roundcube/roundcubemail/releases)
and runs its `installto.sh` update script. The temporary download directory is removed afterward,
and the playbook restores the document root's `apache` ownership.

The playbook uses PHP from [Remi's Safe repository](https://rpms.remirepo.net/), enabled through
Software Collections. Install and configure the repository and the matching PHP SCL (for example,
`php85`) on the target host before running the playbook.

#### Run

```bash
ansible-playbook -i inventory.ini roundcube-update.yml -e roundcube_version=1.6.10
```

#### Variables

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `roundcube_version` | Yes | - | Roundcube release to install, for example `1.6.10`. |
| `php_version` | No | `85` | PHP SCL version used to run the update script; the corresponding Remi PHP package must already be installed. |
| `roundcube_database` | No | `roundcube` | Database to dump before the update. |
| `roundcube_directory` | No | `/var/www/roundcube/` | Existing Roundcube document root. |
| `roundcube_backup_directory` | No | `/opt/backup/` | Directory for the document-root archive and database dump. |

