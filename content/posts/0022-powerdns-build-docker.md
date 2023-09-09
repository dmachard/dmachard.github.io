---
title: "Build PowerDNS server with docker"
summary: "Developer guide to build pdns authoritary"
date: 2021-06-24T00:00:00+01:00
draft: false
tags: ['dns', 'powerdns', 'docker']
---

# Build PowerDNS server with docker

clone the source code from powerdns repositories

```bash
git clone https://github.com/PowerDNS/pdns.git
git clone https://github.com/PowerDNS/pdns-builder.git
cp -rf pdns-builder /pdns/builder
```

build with docker

```bash
cd pdns/
sudo docker build . --file Dockerfile-auth --no-cache -t dmachard/pdns-auth-dev
```

prepare pdns config

```bash
mkdir pdns-dev
touch pdns.conf
```

add the following content to your config file

```bash
enable-lua-records=yes
dnsupdate=yes
loglevel=9

webserver=yes
webserver-address=0.0.0.0
webserver-allow-from=0.0.0.0/0
webserver-password=hello
webserver-port=8081
```

run the new pdns 

```bash
sudo docker run -d -p 5354:53/udp -p 5354:53/tcp --name=powerdns-dev --volume=$PWD/pdns.conf:/etc/powerdns/pdns.d/pdns.conf:ro dmachard/pdns-auth-dev
```

add some dns data in your dns

```bash
sudo docker exec powerdns pdnsutil create-zone zone.test ns1.zone.test
sudo docker exec powerdns pdnsutil add-record zone.test ns1 A 3600 128.0.0.1
sudo docker exec powerdns pdnsutil add-record zone.test a A 300 128.0.0.2 
```

test your server

```bash
dig @127.0.0.1 -p 5354 a.zone.test
```
