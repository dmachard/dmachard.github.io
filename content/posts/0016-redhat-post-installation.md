---
title: "RedHat Family 8.x post installation"
date: 2020-06-21T00:00:00+01:00
draft: false
tags: ['system', 'redhat']
---

Post installation tunning for RedHat system family.

## keyboard

```bash
localectl set-keymap fr
```

To confirm your permanent keymap settings execute the localectl command without any arguments


## hostname configuration

```bash
hostnamectl set-hostname <mon nom de machine>
```

## date, time and timezone

```bash
timedatectl set-timezone Europe/Paris
timedatectl set-ntp yes/no
timedatectl set-time [YYYY-MM-DD]
timedatectl set-time [HH:MM:SS]
timedatectl set-local-rtc yes/no
```

### minimal network configuration

```bash
/etc/sysconfig/network-scripts/ifcfg-enp0s3

ONBOOT=yes
BOOTPROTO=static
PREFIX=24
IPADDR=192.168.1.100
GATEWAY=192.168.1.1
DNS1=192.168.1.1

reboot
```

### install minimal tools

```bash
yum install curl vim
```
