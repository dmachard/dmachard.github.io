---
title: "Prometheus installation guide"
date: 2020-12-20T00:00:00+01:00
draft: false
tags: ['docker', 'prometheus']
---

This post details how to run [prometheus](https://prometheus.io/) container

# Configuration

```bash
cat /var/prometheus/prometheus.yml 
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: 'dnstap_receiver'
    basic_auth:
      username: admin
      password: changeme
    static_configs:
      - targets: ['10.0.0.249:8082']
```

# Start prometheus

```bash
docker run -u root -d -p 9090:9090 -v /var/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml --restart=always --name promserver01 prom/prometheus
```

# Test

The web interface is available at http://yourip:9090
