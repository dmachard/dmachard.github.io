---
title: "Installation d'un cluster docker swarm"
summary: "Procédure pour installer un clusteur docker swarm sur 3 machines."
date: 2019-02-09T00:00:00+01:00
draft: false
tags: ['docker', 'swarm']
---

# Installation d'un cluster docker swarm

Procédure pour installer un clusteur docker swarm sur 3 machines.

En prerequis, installer docker-ce sur l'ensemble des machines, le port tcp/2377 doit être ouvert.

sur le manager

```bash
docker swarm init --advertise-addr IP_MANAGER
```

sur le worker

```bash
docker swarm join --token SWMTKN-1-1xgbk6rfab....k8048s IP_MANAGER:2377
```

sur manager ou worker

```bash
docker info
```

sur un manager

```bash
docker node ls
```
