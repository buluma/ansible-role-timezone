# Ansible role [timezone](https://galaxy.ansible.com/ui/standalone/roles/buluma/timezone/documentation)

Ansible Role for Timezone Management

|GitHub|Version|Issues|Pull Requests|Downloads|
|------|-------|------|-------------|---------|
|[![github](https://github.com/buluma/ansible-role-timezone/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-timezone/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-timezone.svg)](https://github.com/buluma/ansible-role-timezone/releases/)|[![Issues](https://img.shields.io/github/issues/buluma/ansible-role-timezone.svg)](https://github.com/buluma/ansible-role-timezone/issues/)|[![PullRequests](https://img.shields.io/github/issues-pr-closed-raw/buluma/ansible-role-timezone.svg)](https://github.com/buluma/ansible-role-timezone/pulls/)|[![Ansible Role](https://img.shields.io/ansible/role/d/buluma/timezone)](https://galaxy.ansible.com/ui/standalone/roles/buluma/timezone/documentation)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/buluma/ansible-role-timezone/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: yes

  roles:
    - role: buluma.timezone
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/buluma/ansible-role-timezone/blob/master/molecule/default/prepare.yml):

```yaml
---

- name: Prepare
  hosts: all
  remote_user: root
  become: true
  gather_facts: false
  tasks:
    - name: Redhat | Subscription-manager register
      ansible.builtin.raw: |
        set -eu
        subscription-manager register \
          --username={{ lookup('env', 'REDHAT_USERNAME') }} \
          --password={{ lookup('env', 'REDHAT_PASSWORD') }} \
          --auto-attach
      changed_when: false
      failed_when: false

- name: Prepare all
  hosts: all
  remote_user: root
  become: true
  tasks:
    - name: CP -rfT /etc/skel /root
      ansible.builtin.raw: |
        set -eu
        cp -rfT /etc/skel /root
      changed_when: false
      failed_when: false

    - name: Setenforce 0
      ansible.builtin.raw: |
        set -eu
        setenforce 0
        sed -i 's/^SELINUX=.*$/SELINUX=permissive/g' /etc/selinux/config
      changed_when: false
      failed_when: false

    - name: Systemctl stop firewalld.service
      ansible.builtin.raw: |
        set -eu
        systemctl stop firewalld.service
        systemctl disable firewalld.service
      changed_when: false
      failed_when: false

    - name: Systemctl stop ufw.service
      ansible.builtin.raw: |
        set -eu
        systemctl stop ufw.service
        systemctl disable ufw.service
      changed_when: false
      failed_when: false

    - name: Debian | Apt-get install *.deb
      ansible.builtin.raw: |
        set -eu
        DEBIAN_FRONTEND=noninteractive apt-get install -y bzip2 ca-certificates curl gcc gnupg gzip hostname iproute2 passwd procps python3 python3-apt python3-jmespath python3-lxml python3-pip python3-setuptools python3-venv python3-virtualenv python3-wheel rsync sudo tar unzip util-linux xz-utils zip
      when: ansible_os_family | lower == "debian"
      changed_when: false
      failed_when: false

    - name: Fedora | Yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-dnf-plugin-versionlock python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-utils zip
      when: ansible_distribution | lower == "fedora"
      changed_when: false
      failed_when: false

    - name: Redhat-9 | Yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        yum-config-manager --enable crb || echo $?
        yum-config-manager --enable codeready-builder-beta-for-rhel-9-x86_64-rpms || echo $?
        yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-dnf-plugin-versionlock python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-utils zip
      when: ansible_os_family | lower == "redhat" and ansible_distribution_major_version | lower == "9"
      changed_when: false
      failed_when: false

    - name: Redhat-8 | Yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-dnf-plugin-versionlock python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-utils zip
      when: ansible_os_family | lower == "redhat" and ansible_distribution_major_version | lower == "8"
      changed_when: false
      failed_when: false

    - name: Redhat-7 | Yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        subscription-manager repos --enable=rhel-7-server-optional-rpms || echo $?
        yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-plugin-versionlock yum-utils zip
      when: ansible_os_family | lower == "redhat" and ansible_distribution_major_version | lower == "7"
      changed_when: false
      failed_when: false

    - name: Suse | Zypper -n install *.rpm
      ansible.builtin.raw: |
        set -eu
        zypper -n install -y bzip2 ca-certificates curl gcc gpg2 gzip hostname iproute2 procps python3 python3-jmespath python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow sudo tar unzip util-linux xz zip
      when: ansible_os_family | lower == "suse"
      changed_when: false
      failed_when: false
```

Also see a [full explanation and example](https://buluma.github.io/how-to-use-these-roles.html) on how to use these roles.

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/buluma/ansible-role-timezone/blob/master/defaults/main.yml):

```yaml
---
# Set the system time zone to the specified value.
timezone: "Africa/Nairobi"
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/buluma/ansible-role-timezone/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | Version |
|-------------|--------|--------|
|[buluma.bootstrap](https://galaxy.ansible.com/buluma/bootstrap)|[![Ansible Molecule](https://github.com/buluma/ansible-role-bootstrap/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-bootstrap/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-bootstrap.svg)](https://github.com/shadowwalker/ansible-role-bootstrap)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://buluma.github.io/) for further information.

Here is an overview of related roles:

![dependencies](https://raw.githubusercontent.com/buluma/ansible-role-timezone/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/buluma):

|container|tags|
|---------|----|
|[Kali](https://hub.docker.com/repository/docker/buluma/kali/general)|all|
|[Ubuntu](https://hub.docker.com/repository/docker/buluma/ubuntu/general)|all|
|[EL](https://hub.docker.com/repository/docker/buluma/enterpriselinux/general)|all|
|[Debian](https://hub.docker.com/repository/docker/buluma/debian/general)|all|
|[Fedora](https://hub.docker.com/repository/docker/buluma/fedora/general)|all|

The minimum version of Ansible required is 2.12, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/buluma/ansible-role-timezone/issues)

## [Changelog](#changelog)

[Role History](https://github.com/buluma/ansible-role-timezone/blob/master/CHANGELOG.md)

## [License](#license)

[Apache-2.0](https://github.com/buluma/ansible-role-timezone/blob/master/LICENSE)

## [Author Information](#author-information)

[Shadow Walker](https://buluma.github.io/)

