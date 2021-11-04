---
title: "Recursor PowerDNS installation on CentOS"
date: 2020-06-21T00:00:00+01:00
draft: false
tags: ['dns', 'powerdns', 'recursor']
---

PowerDNS recursor servers installation on CentOS.

## Installation

```bash
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum -y install epel-release
yum --enablerepo=epel install luajit libsodium
yum --enablerepo=epel install jq

curl -o /etc/yum.repos.d/powerdns-rec-43.repo https://repo.powerdns.com/repo-files/centos-rec-43.repo
yum install pdns-recursor pdns-tools
```

## Recursor server configuration

```bash
/etc/pdns-recursor/recursor.conf

local-address=0.0.0.0
local-port=53
security-poll-suffix=
forward-zones=home.local=127.0.0.1:5300
forward-zones-recurse=.=10.0.0.140:53
```

## Start servers

```bash
systemctl enable pdns-recursor.service
systemctl start pdns-recursor.service
```

## Manage configuration with rec_control command

## Cache purge entries 

```bash
rec_control wipe-cache test1.home.local
```
