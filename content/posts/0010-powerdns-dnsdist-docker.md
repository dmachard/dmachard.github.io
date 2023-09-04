---
title: "How to run pdns-auth in a Docker Container with custom configuration file"
summary: "This post details how to execute pdns-auth in docker container"
date: 2021-12-22T00:00:00+01:00
draft: false
tags: ['dns', 'powerdns', 'auth', 'docker']
---

# How to run pdns-auth in a Docker Container with custom configuration file

The `pdns-auth` product is available in the official [dockerhub registry](https://hub.docker.com/u/powerdns) of PowerDNS.
This post details how to execute pdns-auth in **docker container**, **custom configuration file** with **sqlite3** database. 
We assume you have a containers environnement already available.

## Custom config

```bash
local-address=0.0.0.0
local-port=53

launch=gsqlite3
gsqlite3-database=/var/lib/powerdns/pdns.sqlite3

dnsupdate=yes
enable-lua-records=yes
```

## Deploy container

Deploy the dnsdist image with the custom configuration

```bash
docker run -d -p 53:53/udp -p 53:53/tcp --restart unless-stopped --name=pdns01 \
--volume=$PWD/pdns.conf:/etc/powerdns/pdns.conf:z powerdns/pdns-auth-45:4.5.2
```

## Docker composer

```bash
version: "3"
services:
  pdns:
    image: powerdns/pdns-auth-45:4.5.2
    ports:
      - mode: host
        protocol: udp
        published: 53
        target: 53
      - mode: host
        protocol: tcp
        published: 53
        target: 53
    user: "1000:1000"
    volumes:
      - ${APP_CONFIG}/pdns/pdns.conf:/etc/powerdns/pdns.conf
      - ${PDNS_STORAGE}/run:/var/run/pdns
      - ${PDNS_STORAGE}/db:/var/lib/powerdns
```

## Persistent database 

Download database schema for sqlite3

```bash
wget https://raw.githubusercontent.com/PowerDNS/pdns/rel/auth-4.5.x/modules/gsqlite3backend/schema.sqlite3.sql
```

Create the database

```bash
sqlite3 pdns.sqlite3 < schema.sqlite3.sql
```

## Create zone

```bash
pdnsutil create-zone <dnszone> ns1.<dnszone>
pdnsutil add-record <dnszone> ns1 A 3600 192.168.1.221
```

## Enable DNS Update

```bash
pdnsutil generate-tsig-key tsigkey hmac-sha256
pdnsutil set-meta <dnszone> TSIG-ALLOW-DNSUPDATE tsigkey
pdnsutil set-meta <dnszone> TSIG-ALLOW-AXFR tsigkey
pdnsutil set-meta <dnszone> ALLOW-DNSUPDATE-FROM 0.0.0.0/0
```

Test with nsupdate

```bash
touch dnsupdate_add.txt
server <dns_ip_server>
zone <dnszone>
update add dnsupdate.<dnszone>. 3600 A 10.10.10.10
show
send
```
 
```bash
touch dnsupdate_del.txt
server <dns_ip_server>
zone <dnszone>
update del dnsupdate.<dnszone>. 3600 A 10.10.10.10
show
send
```

Create record

```bash
nsupdate -p 53 -v -y hmac-sha256:tsigkey:$TSIGKEY -v dnsupdate_add.txt
```