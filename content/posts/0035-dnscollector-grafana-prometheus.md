---
title: "DnsCollector/dnstap: collect dnstap stream and get statistics usage"
date: 2021-11-10T00:00:00+01:00
draft: false
tags: ['dnstap', 'logs', 'prometheus', 'grafana']
---

Example to collect dnstap stream and get statistics usage 

# Table of contents

* [Prequisites](#prequisites)
* [Overview](#overview)
* [Configuration](#configuration)
* [Dashboard](#dashboard)

# Prequisites

Install the dnscollector like described in the following [guide](https://dmachard.github.io/posts/0007-dnscollector-install-binary/).

# Overview

With this example the collector waits incoming dnstap messages sent by your dns server, then you can watch statistics and metrics on your Grafana dashboard.

![prometheus dnscollector](/images/0035/overview.png)


# Configuration

Download the [config.yml](https://github.com/dmachard/go-dnscollector/blob/main/example-config/use-case-2.yml) file. 

```
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
  webserver:
    enable: true
    listen-ip: 0.0.0.0
    listen-port: 8080
    basic-auth-login: admin
    basic-auth-pwd: changeme
    tls-support: true
    cert-file: "/etc/dnscollector/dnscollector.crt"
    key-file: "/etc/dnscollector/dnscollector.key"
```

# Dashboard

The dashboard can be found [here](https://github.com/dmachard/go-dnscollector/blob/main/example-config/grafana-dashboard.json).

A small overview 

![dashboard dnscollector](/images/0035/dashboard.png)
