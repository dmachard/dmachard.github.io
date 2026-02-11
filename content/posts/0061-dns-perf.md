---
title: "DNS Performance Testing Tools"
summary: "DNS tools for testing performance purpose"
date: 2024-09-12T00:00:00+01:00
draft: false
tags: ['dns', 'tools', 'flamethrower', 'dnsperf']
pin: false
---

# DNS Performance Testing Tools

This setup helps you benchmark DNS performance under different load scenarios, ensuring optimal server configuration and responsiveness.

## Framethrower

[Flamethrower](https://github.com/DNS-OARC/flamethrower) is a flexible and fast DNS performance and load testing tool. It supports IPv4/IPv6 and both UDP and TCP, making it ideal for various test scenarios. You can easily run Flamethrower using Docker:

```bash
sudo docker run ns1labs/flame:0.12.0-master -p <TARGET_PORT> -Q <QPS> <TARGET_IP>
```

* -p <TARGET_PORT>: Specify the target port (usually 53 for DNS).
* -Q <QPS>: Queries per second, set the desired query load.
* <TARGET_IP>: Target DNS server IP.

## DNSperf

[DNSperf](https://github.com/DNS-OARC/dnsperf) is another widely used tool for DNS performance testing. It allows you to generate large volumes of DNS queries and measure response times. Similar to Flamethrower, you can also run DNSperf in Docker.

```bash
sudo docker run ns1labs/flame:0.12.0-master -p <TARGET_PORT> -Q <QPS> <TARGET_IP>
```