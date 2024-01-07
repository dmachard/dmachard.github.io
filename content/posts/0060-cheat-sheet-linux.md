---
title: "My inux cheat sheet"
summary: "My Linux cheat sheet"
date: 2024-01-07T00:00:00+01:00
draft: false
tags: ['cheat-sheet']
pin: false
---

# My linux cheat sheet

| Actions                             | Description |
|-------------------------------------|-------------|
| show version ubuntu                 | `lsb_release -a` |
| add static ip with netplan           | ```yaml
                                        $ sudo vim /etc/netplan/01-cfg-ens19.yaml<br/>
                                        network:<br/>
                                          ethernets:<br/>
                                            ens19:<br/>
                                              addresses:<br/>
                                              - 172.16.0.1/12<br/>
                                          version: 2<br/>
                                        sudo chmod 600 /etc/netplan/*<br/>
                                        sudo netplan apply
                                        ``` |
| authenticate with ssh certificate   | ```bash
                                        $ ssh-keygen<br/>
                                        $ ssh-copy-id username@remote_host
                                        ``` |

{{< bootstrap-table "table table-dark table-striped table-bordered" >}}
| Animal  | Sounds |
|---------|--------|
| Cat     | Meow   |
| Dog     | Woof   |
| Cricket | Chirp  |
{{< /bootstrap-table >}}