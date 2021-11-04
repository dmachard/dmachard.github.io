---
title: "Initial Ansible setup server"
date: 2021-10-23T00:00:00+01:00
draft: false
tags: ['ansible', 'system']
---

This tutorial explains how to make the initial setup of ansible.

On this tutorial we assume you have at least python installed on all servers to manage and all your servers are available with a hostname in your dns and a root ssh access.

Create a specific automation user for your ansible srever 

```bash
adduser automation
passwd automation
echo  -e 'automation\tALL=(ALL)\tNOPASSWD:\tALL' > /etc/sudoers.d/automation
```

Connect with-it

```bash
su - automation
```

Create key pair

```bash
ssh-keygen -o -b 4096
```

Describe your inventory.

```bash
vim inventory.ini 
[group1]
server01
server02
server03
```

Create default ansible configuration and define the python path.

touch ansible.cfg

```bash
[defaults]
interpreter_python=/usr/bin/python3
```

Run the *playbook_deploy_ansible_user.yml* playblook to configure:
- a specific user for ansible to run
- disable root ssh access
- disable password authentication

Run-it with the root account of each server, after that the connection to the server can be done with the automation user.

```bash
$ ansible-playbook -i inventory.ini setup-ansible/playbook.yml --ask-pass -u root
SSH password: 

PLAY [all] *************************************************************************

TASK [Gathering Facts] *************************************************************
ok: [server01]
ok: [server02]
ok: [server03]

TASK [Add a new user named automation] *********************************************
ok: [server03]
ok: [server01]
ok: [server02]

TASK [Add automation user to the sudoers] ******************************************
changed: [server03]
changed: [server02]
changed: [server01]

TASK [Deploy SSH Key] **************************************************************
changed: [server01]
changed: [server02]
changed: [server03]

TASK [Disable Password Authentication] *********************************************
changed: [server01]
changed: [server03]
changed: [server02]

TASK [Disable Root Login] **********************************************************
changed: [server02]
changed: [server03]
changed: [server01]

RUNNING HANDLER [restart_ssh] ******************************************************
changed: [server01]
changed: [server02]
changed: [server03]

PLAY RECAP *************************************************************************
server01    : ok=7    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
server02    : ok=7    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
server03    : ok=7    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Finally run the ansible command with ping module

```bash
$ ansible all -i inventory.ini -m ping
server01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
server02 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
server03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```