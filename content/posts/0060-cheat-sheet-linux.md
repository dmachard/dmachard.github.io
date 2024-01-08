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

{{< bootstrap-table "table table-striped table-bordered" >}}
| Animal  | Sounds |
|---------|--------|
| Cat     | Meow   |
| Dog     | ```yaml
            $ sudo vim /etc/netplan/01-cfg-ens19.yaml<br/>
            network:<br/>
                ethernets:<br/>
                ens19:<br/>
                    addresses:<br/>
                    - 172.16.0.1/12<br/>
                version: 2<br/>
            sudo chmod 600 /etc/netplan/*<br/>
            sudo netplan apply
            ```   |
| Cricket | Chirp  |
{{< /bootstrap-table >}}


| Status | Response  |
| ------ | --------- |
| 200    |<pre lang="json">{<br>  "id": 10,<br>  "username": "alanpartridge",<br>  "email": "alan@alan.com",<br>  "password_hash": "$2a$10$uhUIUmVWVnrBWx9rrDWhS.CPCWCZsyqqa8./whhfzBZydX7yvahHS",<br>  "password_salt": "$2a$10$uhUIUmVWVnrBWx9rrDWhS.",<br>  "created_at": "2015-02-14T20:45:26.433Z",<br>  "updated_at": "2015-02-14T20:45:26.540Z"<br>}</pre>|
| 400    |<code>{<br>  "code": 400,<br>  "msg": balabala"<br>}</code>|
