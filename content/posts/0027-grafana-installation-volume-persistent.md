---
title: "Grafana installation with persistent volume"
date: 2019-01-29T00:00:00+01:00
draft: false
tags: ['docker', 'installation']
---

This post details how to run grafana container with persistent volume

# Table of contents

* [Create volume](#create-volume)
* [Start Grafana](#start-grafana)
* [Test](#test)

# Create volume

Create a persistent volume for your data in /var/lib/grafana (database and plugins)

```bash
docker volume create grafana-storage
```

# Start grafana

```bash
docker run  -d -p 80:3000 --name=grafana -v grafana-storage:/var/lib/grafana grafana/grafana
```

# Test

http://yourip:80/

Default account:
- login: admin
- password: admin