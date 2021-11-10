---
title: "DnsCollector/dnstap: collect dnstap stream and backup-it to log files"
date: 2021-10-10T00:00:00+01:00
draft: false
tags: ['dnstap', 'logs']
---

Example to collect dnstap messages from dns servers and backup-it in log files


# Table of contents

* [Prequisites](#prequisites)
* [Configuration](#configuration)
* [Logs](#logs)

# Prequisites

Install the dnscollector like described in the following [guide](https://dmachard.github.io/posts/0007-dnscollector-install-binary/).

# Configuration

Download the [config.yml](https://github.com/dmachard/go-dnscollector/blob/main/example-config/use-case-1.yml) file. 
With this configuration, the collector waits incoming dnstap messages and save-it in log files in text format with rotation.

```yaml
trace:
  verbose: true

collectors:
  dnstap:
    enable: true
    listen-ip: 0.0.0.0
    listen-port: 6000

loggers:
  logfile:
    enable: true
    file-path:  "/var/run/dnscollector/dnstap.log"
    max-size: 100
    max-files: 10
    mode: text
```

# Logs

```bash
tail -f /var/run/dnscollector/dnstap.log
```