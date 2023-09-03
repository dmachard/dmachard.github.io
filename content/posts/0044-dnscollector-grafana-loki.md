---
title: "Collect DNSTAP stream and analysing DNS logs with Loki and Grafana"
summary: "Example to collect dnstap stream and analysing logs with Loki and Grafana"
date: 2021-12-16T00:00:00+01:00
draft: false
tags: ['dnstap', 'logs', 'loki', 'grafana', 'go-dnscollector']
pin: true
---

# Collect DNSTAP stream and analysing DNS logs with Loki and Grafana

Example to collect dnstap stream and analysing logs with Loki+Grafana

# Prequisites

Install the dnscollector like described in the following [guide](https://dmachard.github.io/posts/0007-dnscollector-install-binary/).

# Overview

With this example the collector waits incoming dnstap messages sent by your dns server, then you can watch and analysing logs on your Grafana dashboard.

![prometheus dnscollector](/images/0044/use-case-4.png)

# Configuration

Download the [config.yml](https://github.com/dmachard/go-dnscollector/blob/main/example-config/use-case-4.yml) file. 

```ini
global:
  trace:
    verbose: true

multiplexer:
  collectors:
    - name: tap
      dnstap:
        listen-ip: 0.0.0.0
        listen-port: 6000
        tls-support: true
        cert-file: "/etc/dnscollector/dnscollector.crt"
        key-file: "/etc/dnscollector/dnscollector.key"

  loggers:
    - name: loki
      lokiclient:
        server-url: "http://loki:3100/loki/api/v1/push"
        job-name: "dnscollector"
        text-format: "localtime identity qr queryip family protocol qname qtype rcode"

  routes:
    - from: [tap]
      to: [loki]
```

# Dashboard

The dashboard can be found [here](https://github.com/dmachard/grafana-dashboards/tree/main/Go-DnsCollector).

A small overview 

![dashboard dnscollector](/images/0044/dashboard.png)
