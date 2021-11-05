---
title: "Freshrss installation guide in container mode"
date: 2021-03-10T00:00:00+01:00
draft: true
tags: ['docker', 'freshrss']
---

This post details how to run [freshrss](https://freshrss.org/) container

# Create volumes

```bash
docker volume create freshrss-data
docker volume create freshrss-extensions
```

# Start freshrss

```bash
docker run -d --restart unless-stopped --log-opt max-size=10m   -v freshrss-data:/var/www/FreshRSS/data   -v freshrss-extensions:/var/www/FreshRSS/extensions   -e 'CRON_MIN=4,34'   -e TZ=Europe/Paris   -p 8080:80   --name freshrss freshrss/freshrss
```