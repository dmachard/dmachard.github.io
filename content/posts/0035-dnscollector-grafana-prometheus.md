---
title: "Insights into your DNS traffic with DNS-collector"
summary: "Overview of the DNS-collector tool, an open-source DNS data collector"
date: 2021-11-10T00:00:00+01:00
draft: false
tags: ['dnstap', 'logs', 'prometheus', 'dnscollector']
pin: true
---

# Insights into your DNS traffic with DNS-collector

## What is DNS-collector?

![DNS-collector](https://raw.githubusercontent.com/dmachard/go-dnscollector/main/docs/dns-collector_logo.png)

[DNS-collector](https://github.com/dmachard/go-dnscollector) is an open-source DNS data collector written in Go started sinc august 2021. It acts as a passive high speed ingestor, aggregator and distributor for logs with usage indicators and security analysis.

DNS-collector can collect and aggregate DNS traffic from simultaneously sources like [DNStap](https://dnstap.info/) streams, network interface or log files and relays it to multiple other listeners with some transformations on it (traffic filtering, user privacy, ...).

![ELK dashboard image](/images/0035/overview.png)

## How to deploy DNS-collector with Docker

> Before proceeding, please follow the guide [Enabling DNStap logging on most popular DNS servers](https://dmachard.github.io/posts/0001-dnstap-testing/) to ensure that your DNS traffic is sent to the collector.

Create the `config.yml` file

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
    - name: console
      stdout:
        mode: flatjson
    - name: prom
      prometheus:
        listen-ip: 0.0.0.0
        listen-port: 8081
 
  routes:
    - from: [ tap ]
      to: [ console, prometheus ]

```

> You can find various [examples](https://github.com/dmachard/go-dnscollector#usage-examples) on the github project.

Pull the image and start the container

```bash
sudo docker run -d -v $(pwd)/config.yml:/etc/dnscollector/config.yml --name dnscollector -p 6000:6000/tcp dmachard/go-dnscollector:latest
```

Display the logs

```bash
sudo docker logs dnscollector
```

You will observe output similar to the following:

```json
{
  "dns.flags.aa":false,
  "dns.flags.ad":true,
  "dns.flags.qr":false,
  "dns.flags.ra":false,
  "dns.flags.tc":false,
  "dns.length":55,
  "dns.malformed-packet":false,
  "dns.opcode":0,
  "dns.qname":"www.google.com",
  "dns.qtype":"A",
  "dns.rcode":"NOERROR",
  "dns.resource-records.an":[],
  "dns.resource-records.ar":[],
  "dns.resource-records.ns":[],
  "dnstap.extra":"-",
  "dnstap.identity":"dnsdist",
  "dnstap.latency":"0.000000",
  "dnstap.operation":"CLIENT_QUERY",
  "dnstap.timestamp-rfc3339ns":"2023-09-27T17:03:54.651904447Z",
  "dnstap.version":"dnsdist 1.8.1",
  "edns.dnssec-ok":0,
  "edns.options.0.code":10,
  "edns.options.0.data":"-",
  "edns.options.0.name":"COOKIE",
  "edns.rcode":0,
  "edns.udp-size":1232,
  "edns.version":0,
  "network.family":"IPv4",
  "network.ip-defragmented":false,
  "network.protocol":"UDP",
  "network.query-ip":"172.17.0.1",
  "network.query-port":"49458",
  "network.response-ip":"172.17.0.2",
  "network.response-port":"53",
  "network.tcp-reassembled":false
}
```

## Visualizing usage indicators and logs from Loki and Grafana

### Configure prometheus

Configure Prometheus to scrape metrics from DNS-collector:

```yaml
  - job_name: 'dnscollector'
    scrape_interval: 5s
    basic_auth:
      username: 'admin'
      password: 'changeme'
    static_configs:
      - targets: [ dnscollector:8080 ]
```

### Loki

For tracing your logs in [Loki](https://grafana.com/oss/loki/), a [build-in](https://grafana.com/grafana/dashboards/15415) dashboard is available

![loki dashboard image](/images/0035/dashboard_loki.png)

### Grafana

Additionally, a [build-in](https://grafana.com/grafana/dashboards/16630) Grafana dashboard is provided to get usage indicators.

![grafana dashboard image](/images/0035/dashboard_prometheus.png)
