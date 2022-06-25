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
global:
  trace:
    verbose: false

multiplexer:
  collectors:
    - name: tap
      dnstap:
        listen-ip: 0.0.0.0
        listen-port: 6000

  loggers:
    - name: std_out
      stdout:
        mode: json

  routes:
    - from: [tap]
      to: [std_out]
```

# Logs

```bash
tail -f /var/run/dnscollector/dnstap.log | jq
```

```json
{
  "network": {
    "family": "INET",
    "protocol": "UDP",
    "query-ip": "192.168.1.200",
    "query-port": "53114",
    "response-ip": "172.18.0.8",
    "response-port": "53",
    "as-number": "-",
    "as-owner": "-"
  },
  "dns": {
    "length": 178,
    "opcode": 0,
    "rcode": "NOERROR",
    "qname": "v10.events.data.microsoft.com",
    "qtype": "A",
    "flags": {
      "qr": true,
      "tc": false,
      "aa": false,
      "ra": true,
      "ad": false
    },
    "resource-records": {
      "an": [
        {
          "name": "v10.events.data.microsoft.com",
          "rdatatype": "CNAME",
          "ttl": 3595,
          "rdata": "global.asimov.events.data.trafficmanager.net"
        },
        {
          "name": "global.asimov.events.data.trafficmanager.net",
          "rdatatype": "CNAME",
          "ttl": 55,
          "rdata": "onedscolprdweu00.westeurope.cloudapp.azure.com"
        },
        {
          "name": "onedscolprdweu00.westeurope.cloudapp.azure.com",
          "rdatatype": "A",
          "ttl": 5,
          "rdata": "13.69.109.130"
        }
      ],
      "ns": [],
      "ar": []
    },
    "malformed-packet": 0
  },
  "edns": {
    "udp-size": 0,
    "rcode": 0,
    "version": 0,
    "dnssec-ok": 0,
    "options": []
  },
  "dnstap": {
    "operation": "CLIENT_RESPONSE",
    "identity": "29e7b0f3cc19",
    "timestamp-rfc3339ns": "2021-12-29T07:17:36.934966336Z",
    "latency": "0.008425"
  },
  "geo": {
    "city": "-",
    "continent": "-",
    "country-isocode": "-"
  }
}
```


