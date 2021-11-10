---
title: "DnsCollector/dnstap: installation guide from binary"
date: 2021-09-08T00:00:00+01:00
draft: false
tags: ['dnstap', 'installation']
---

This post details how to install the [go-dnscollector](https://github.com/dmachard/go-dnscollector) tool with systemd.

# Table of contents

* [Installation](#installation)
* [Certificate](#certificate)
* [Configuration](#configuration)
* [Start](#start)

# Installation

Install go-dnscollector from binary

Create some folders and user

```bash
adduser -M dnscollector

mkdir /etc/dnscollector/
mkdir /var/run/dnscollector/
```

Export the following variable to get the version you want to download

```bash
export DNSCOLLECTOR_VERSION=v0.10.0
```

Download the binary

```bash
wget https://github.com/dmachard/go-dnscollector/releases/download/$(echo $DNSCOLLECTOR_VERSION)/go-dnscollector_$(echo $DNSCOLLECTOR_VERSION)_linux_amd64.tar.gz
tar xvf go-dnscollector_$(echo $DNSCOLLECTOR_VERSION)_linux_amd64.tar.gz
mv go-dnscollector /usr/bin/
mv config.yml /etc/dnscollector/config.yml.default
```

# Certificate

Create a certificate. In this example we used a self-signed cert. Prefer to use an official TLS certificate according to your context.

```bash
cd /etc/dnscollector/
openssl req -x509 -nodes -newkey rsa:2048 -keyout dnscollector.key -out dnscollector.crt
```

# Configuration

Configure go-dnscollector

```bash
touch /etc/dnscollector/config.yml

vim config.yml
trace:
  verbose: true

collectors:
  dnstap:
    enable: true
    listen-ip: 0.0.0.0
    listen-port: 6000
    tls-support: true
    cert-file: "/etc/dnscollector/dnscollector.crt"
    key-file: "/etc/dnscollector/dnscollector.key"

loggers:
  logfile:
    enable: true
    file-path:  "/var/run/dnscollector/dnstap.log"
    max-size: 100
    max-files: 10
    mode: text
```


# Start

Enable & Start stunnel. Configure your systemd service

```bash
vim /usr/lib/systemd/system/dnscollector.service

[Unit]
Description=Go DnsCollector
Documentation=https://github.com/dmachard/go-dnscollector
Wants=network-online.target
After=network-online.target

[Service]
User=dnscollector
Group=dnscollector
ExecStart=/usr/bin/go-dnscollector --config /etc/dnscollector/config.yml
ExecStop=/usr/bin/pkill go-dnscollector
Type=simple

[Install]
WantedBy=multi-user.target
```

Chown

```bash
chown dnscollector:dnscollector -R /etc/dnscollector/
chown -R dnscollector:dnscollector /var/run/dnscollector/
```

Enable and start the go-dnscollector service.

```bash
systemctl enable --now dnscollector
systemctl restart dnscollector
```
