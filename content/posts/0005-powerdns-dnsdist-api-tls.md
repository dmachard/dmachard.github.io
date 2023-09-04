---
title: "How to configure TLS for the API using stunnel and DNSdist"
date: 2021-09-08T00:00:00+01:00
draft: false
tags: ["dns", 'powerdns', 'dnsdist', 'api', 'tls']
---

# How to configure TLS for the API using stunnel and DNSdist

This post will detail how to wrap your DnsDIST webserver/API and dnstap stream with TLS using **stunnel**.

## Purpose

This tutorial assumes you have a working PowerDNS dnsdist server installed on a Centos/AlmaLinux with webserver api. Also we will use the same user/group that dnsdist for stunnel. 
Any feedbacks will be appreciated to improve this tutorial.

## Installation

Install stunnel

```bash
yum install stunnel
mkdir /var/run/stunnel
chown dnsdist:dnsdist /var/run/stunnel
```
## Configuration

Create a certificate. In this example we used a self-signed cert. Prefer to use an official TLS certificate according to your context.

```bash
cd /etc/stunnel/
openssl req -x509 -nodes -newkey rsa:2048 -keyout stunnel.key -out stunnel.crt

```

Replace the key <your_dnstap_collector> by your [dnstap collector](https://github.com/dmachard/go-dnscollector) address.

```bash
vim /etc/stunnel/stunnel.conf

chroot = /var/run/stunnel
setuid = dnsdist
setgid = dnsdist
pid = /stunnel.pid
socket = l:TCP_NODELAY=1
socket = r:TCP_NODELAY=1

ciphers=EECDH+AESGCM:EDH+AESGCM
sslVersion = TLSv1.2
options = NO_SSLv2
options = NO_SSLv3

[dnsdist-webapi]
accept=443
connect=127.0.0.1:8080
cert = /etc/stunnel/stunnel.crt
key = /etc/stunnel/stunnel.key
```

## Systemd

Enable & Start stunnel and configure your systemd service

```bash
vim /usr/lib/systemd/system/stunnel.service

[Unit]
Description=TLS tunnel for network daemons
After=dnsdist.target

[Service]
ExecStart=/usr/bin/stunnel /etc/stunnel/stunnel.conf
ExecStop=/usr/bin/pkill stunnel
Type=forking

[Install]
WantedBy=multi-user.target
```

Enable and start the stunnel service.

```bash
systemctl enable --now stunnel
systemctl restart stunnel
```

