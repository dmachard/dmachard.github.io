---
title: "My inux cheat sheet"
summary: "My Linux cheat sheet"
date: 2024-01-07T00:00:00+01:00
draft: false
tags: ['cheat-sheet']
pin: false
---

# My linux cheat sheet

| Actions | Commands  |
| ------ | --------- |
| show version ubuntu                 | <code>lsb_release -a</code> |
| add static ip with netplan          |<pre lang="yaml">$ sudo vim /etc/netplan/01-cfg-ens19.yaml<br/>network:<br/>    ethernets:<br/>    ens19:<br/>        addresses:<br/>        - 172.16.0.1/12<br/>    version: 2<br/>sudo chmod 600 /etc/netplan/*<br/>sudo netplan apply</pre>|
| authenticate with ssh certificate   | <code>$ ssh-keygen<br/>$ ssh-copy-id username@remote_host</code> |
