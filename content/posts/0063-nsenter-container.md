---
title: "Accessing a Docker Container's Network Namespace"
summary: "without installing any tools"
date: 2024-12-26T00:00:00+01:00
draft: false
tags: ['container', 'networking']
pin: false
---

This post show how to to execute network commands (like ip a, ping, etc.) within the container's network namespace without installing any tools.

1. Get the Process ID (PID) of the Container. Use the docker inspect command to retrieve the PID of the container. This will output the PID of the container's main process

```bash
sudo docker inspect -f '{{.State.Pid}}' <container_name_or_id>
```

2. Use the nsenter command to enter the network namespace of the container, Replace <PID> with the PID you obtained before.

```bash
sudo nsenter --net=/proc/<PID>/ns/net
```
