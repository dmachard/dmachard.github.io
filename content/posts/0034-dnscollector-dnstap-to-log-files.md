---
title: "DnsCollector/dnstap: collect dnstap stream and backup-it to log files"
date: 2021-11-10T00:00:00+01:00
draft: false
tags: ['dnstap', 'logs']
---

Example to collect dnstap messages from dns servers and backup-it in log files


# Table of contents

* [Prequisites](#prequisites)
* [Overview](#overview)
* [Configuration](#configuration)
* [Logs](#logs)

# Prequisites

Install the dnscollector like described in the following [guide](https://dmachard.github.io/posts/0007-dnscollector-install-binary/).

# Overview


With this example, the collector waits incoming dnstap messages and save-it in log files in text format with rotation.

![overview dnstap](/images/0034/use-case-1.png)

# Configuration

Download the [config.yml](https://github.com/dmachard/go-dnscollector/blob/main/example-config/use-case-1.yml) file. 

```yaml
trace:
  verbose: true

multiplexer:
  collectors:
    - name: "tap"
      dnstap:
        listen-ip: 0.0.0.0
        listen-port: 6000

  loggers:
    - name: file
      logfile:
        file-path:  "/var/run/dnscollector/dnstap.log"
        max-size: 100
        max-files: 10
        mode: text

  routes:
    - from: [tap]
      to: [file]
```

# Logs

You can follow the logs like this:

```bash
tail -f /var/run/dnscollector/dnstap.log
2021-12-12T13:05:55.036223063Z 75ebec340c0c CLIENT_QUERY NOERROR 192.168.1.12 39200 INET UDP 49b ns1-1.akamaitech.net AAAA 0.000000
2021-12-12T13:05:55.036244143Z 75ebec340c0c CLIENT_RESPONSE NOERROR 192.168.1.12 39200 INET UDP 115b ns1-1.akamaitech.net AAAA 0.000021
```