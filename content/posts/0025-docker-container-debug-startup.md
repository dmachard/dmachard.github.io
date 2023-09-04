---
title: "Debug sur container ne démarrant pas"
summary: "Quelques commandes pour debugger un container qui ne démarre pas"
date: 2019-07-18T00:00:00+01:00
draft: false
tags: ['docker', 'debug', 'howto']
---

# Debug sur container ne démarrant pas

Quelques commandes pour debugger un container qui ne démarre pas

## How to

**Comment obtenir un shell dans un container?**

```bash
docker exec -it <container_id> bash
```

**Comment debugguer sur container qui démarre pas?** en modifiant le entrypoint par sh ce qui permet ensuite d'exécuter à la main le script ou programme posant problème.

```bash
docker run -it  --name=<moncontainer> --entrypoint=sh <container_img>
```

**Comment afficher les logs d'un container?**

```bash
docker logs --follow <container_id>
```
