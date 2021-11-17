---
title: "Install a Docker Swarm cluster with Persistent Storage Using GlusterFS"
date: 2021-11-17T00:00:00+01:00
draft: false
tags: ['docker', 'swarm', 'glusterfs', 'installation']
---

Install a Docker Swarm cluster with Persistent Storage Using GlusterFS

# Table of contents

* [Prerequisites](#prerequisites)
* [Overview](#overview)

# Prerequisites

Your ansible inventory and hosts file should be like this:

cat /etc/hosts

```bash
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.221	docker-manager01
192.168.1.222	docker-worker01
192.168.1.223	docker-worker02
```

cat inventory.ini

```bash
[swarm_managers]
docker-manager01

[swarm_workers]
docker-worker01
docker-worker02

[swarm:children]
swarm_managers
swarm_workers

[glusterfs]
docker-manager01
docker-worker01
docker-worker02
```

# Overview

The cluster consists of 3 servers:
- 1 manager
- 2 workers
- A distributed file system

![overview docker swarm and glusterfs](/images/0037/dockerswarm_glusterfs.png)

# Installation

Run the following playbooks

- ansible-playbook [setup_docker/playbook.yml](https://github.com/dmachard/ansible-playbooks/tree/main/setup_docker)
- ansible-playbook [setup_dockerswarm/playbook.yml](https://github.com/dmachard/ansible-playbooks/tree/main/setup_dockerswarm)
- ansible-playbook [setup_glusterfs/playbook.yml](https://github.com/dmachard/ansible-playbooks/tree/main/setup_glusterfs)