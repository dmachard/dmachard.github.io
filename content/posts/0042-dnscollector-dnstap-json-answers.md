---
title: "DnsCollector/dnstap: collect dnstap stream and follow dns answers with JSON format"
date: 2021-11-27T00:00:00+01:00
draft: false
tags: ['dnstap', 'logs']
---

Example to collect dnstap messages from dns servers and follow dns answers with JSON format.

# Table of contents

* [Prequisites](#prequisites)
* [Overview](#overview)
* [Configuration](#configuration)
* [Logs](#logs)

# Prequisites

Install the dnscollector like described in the following [guide](https://dmachard.github.io/posts/0007-dnscollector-install-binary/).

# Overview

With this example, the collector waits incoming dnstap messages and redirect them to stdout in JSON format.
JSON ouput can be used to get dns answers.

# Configuration

Download the [config.yml](https://github.com/dmachard/go-dnscollector/blob/main/example-config/use-case-3.yml) file. 

```yaml
trace:
  verbose: true

collectors:
  dnstap:
    enable: true
    listen-ip: 0.0.0.0
    listen-port: 6000

loggers:
  stdout:
    enable: true
    mode: json
```

# Logs

```bash
tail -f /var/run/dnscollector/dnstap.log
```

```json
{
  "operation": "CLIENT_RESPONSE",
  "identity": "040d0d9c68d8",
  "family": "INET",
  "protocol": "UDP",
  "query-ip": "192.168.1.12",
  "query-port": "52352",
  "response-ip": "172.18.0.4",
  "response-port": "53",
  "length": 127,
  "rcode": "NOERROR",
  "qname": "global.vortex.data.trafficmanager.net",
  "qtype": "AAAA",
  "latency": "0.000013",
  "timestamp-rfc3339": "2021-11-27T14:46:05.471692064Z",
  "answers": null,
  "country-isocode": "-"
}
{
  "operation": "CLIENT_RESPONSE",
  "identity": "040d0d9c68d8",
  "family": "INET",
  "protocol": "UDP",
  "query-ip": "192.168.1.12",
  "query-port": "55441",
  "response-ip": "172.18.0.4",
  "response-port": "53",
  "length": 82,
  "rcode": "NOERROR",
  "qname": "global.vortex.data.trafficmanager.net",
  "qtype": "A",
  "latency": "0.022077",
  "timestamp-rfc3339": "2021-11-27T14:46:05.49368055Z",
  "answers": [
    {
      "name": "global.vortex.data.trafficmanager.net",
      "rdatatype": "A",
      "ttl": 60,
      "rdata": "40.77.226.250"
    }
  ],
  "country-isocode": "-"
}
```


