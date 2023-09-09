---
title: "How to configure TLS for outgoing dnstap stream using stunnel and DNSdist"
summary: "This post will detail how to wrap your outgoing dnstap stream with TLS using stunnel"
date: 2021-09-08T00:00:00+01:00
draft: false
tags: ["dns", 'powerdns', 'dnsdist', 'dnstap', 'security']
---

# How to configure TLS for outgoing dnstap stream using stunnel and DNSdist

This post will detail how to wrap your outgoing dnstap stream with TLS using **stunnel**.

## Introduction

This tutorial assumes you have a working PowerDNS dnsdist server installed on a Centos/AlmaLinux with dnstap enabled. 
Also we will use the same user/group that dnsdist for stunnel. 
Any feedbacks will be appreciated to improve this tutorial.

## Installation

Install stunnel

```bash
yum install stunnel
mkdir /var/run/stunnel
chown dnsdist:dnsdist /var/run/stunnel
```

## Configuration

Configure  stunnel

Replace the key <your_dnstap_collector> by your [dnstap collector](https://github.com/dmachard/go-dnscollector) address.
This is example is done with dnstap unix socket but you can use tcp socket too.

```bash
vim /etc/stunnel/stunnel.conf

chroot = /var/run/stunnel
setuid = dnsdist
setgid = dnsdist
pid = /stunnel.pid
socket = r:TCP_NODELAY=1

[dnsdist-dnstaptls]
client=yes
accept=/var/run/stunnel/dnstap.sock
connect=<your_dnstap_collector>:6000
```

## Systemd

Enable & Start stunnel. Configure your systemd service. All files in /var/run/ are deleted in stop action.

```bash
vim /usr/lib/systemd/system/stunnel.service

[Unit]
Description=TLS tunnel for network daemons
After=dnsdist.target

[Service]
ExecStart=/usr/bin/stunnel /etc/stunnel/stunnel.conf
ExecStop=/usr/bin/pkill stunnel
ExecStop=-/usr/bin/find /var/run/stunnel -mindepth 1 -delete
Type=forking

[Install]
WantedBy=multi-user.target
```

Enable and start the stunnel service.

```bash
systemctl enable --now stunnel
```

