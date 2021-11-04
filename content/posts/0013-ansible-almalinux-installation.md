---
title: "How to install Ansible 4.7 on AlmaLinux 8.4 with pip"
date: 2021-10-20T00:00:00+01:00
draft: false
tags: ['ansible', 'almalinux', 'installation']
---

This post will details how to install ansible on AlmaLinux 8.4.

Tested with the following version:

Ansible 4.7.0
AlmaLinux 8.4
Python 3.9

## Install python 3.9

```
dnf install epel-release -y
dnf install python39 sshpass
```

## Install Ansible

Install some python dependencies

```
pip3 install --upgrade pip
```

Finally install Ansible

```
python3 -m pip install ansible
```

Checking version

```
# ansible --version
ansible [core 2.11.6] 
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.9/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.9.2 (default, May 20 2021, 18:04:00) [GCC 8.4.1 20200928 (Red Hat 8.4.1-1)]
  jinja version = 3.0.2
  libyaml = True

```