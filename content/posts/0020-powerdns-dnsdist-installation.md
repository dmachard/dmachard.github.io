---
title: "Installation guide of DNSdist on CentOS 7"
summary: "Procedure installation"
date: 2020-06-21T00:00:00+01:00
draft: false
tags: ['dns', 'powerdns', 'dnsdist', 'installation']
---

# Installation guide of DNSdist on CentOS 7

## Installation

Activate EPEL repo

```bash
yum install -y epel-release
```

Install dependencies

```bash
yum install re2 net-snmp.x86_64 lmdb.x86_64 fstrm.x86_64
yum install protobuf.x86_64 tinycdb.x86_64 libsodium.x86_64 luajit.x86_64 openssl
yum install gnutls tinycdb
```

Download from the last version available  https://repo.powerdns.com/centos/x86_64/7/

```bash
rpm -ivh dnsdist-1.4.0-1pdns.el7.rpm
```

## Configuration

vim /etc/dnsdist/dnsdist.conf

```lua
-- bind on ip any
setLocal('0.0.0.0:53')

-- allow all IP access
setACL({'0.0.0.0/0'})

-- start the web server on port 8080
webserver("0.0.0.0:8080", "key")

-- disable security feature polling
setSecurityPollSuffix('')

-- log incoming request
addAction(AllRule(), LogAction("/var/log/dnsdist.log", false, true, false, true, true))

-- dns auth list
newServer({address="10.0.0.140", pool="pihole"})

-- accept only dns request
addAction(NotRule(OpcodeRule(DNSOpcode.Update)), RCodeAction(DNSRCode.REFUSED))

-- accept dns request from specific ip or subnest
addAction(NotRule(makeRule("10.0.0.97/32")), RCodeAction(DNSRCode.REFUSED))

-- forward dns request to the autoritative DNS according the zone
addAction(AndRule({QTypeRule(DNSQType.SOA),QNameRule('www.google.fr')}), PoolAction("pihole"))

-- otherwise all DNS requests
addAction(AllRule(), RCodeAction(DNSRCode.REFUSED))
```

## Start 

```bash
systemctl enable dnsdist.service
systemctl start dnsdist.service
systemctl status dnsdist.service
```

## firewalld configuration

```bash
firewall-cmd --permanent --add-port=53/udp
firewall-cmd --permanent --add-port=53/tcp
firewall-cmd --reload
```
