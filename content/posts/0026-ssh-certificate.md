---
title: "Connexion SSH par certificat"
date: 2018-10-17T00:00:00+01:00
draft: false
tags: ['certificat', 'ssh']
---

Ce poste détaille comment se connecter sur un système distant par certificat.

# Génération clé

```bash
# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
c7:da:800:22:9e:a1:1f root@automation
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|     ...  ..o .  |
+-----------------+
```

# Copier la clé publique

Copier la clé publique sur le système distant

```bash
# ssh-copy-id root@192.168.1.235
The authenticity of host '192.168.1.235 (192.168.1.235)' can't be established.
ECDSA key fingerprint is 7c:23:c6:ba:25.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.1.235's password:

Number of key(s) added: 1
```