---
title: "PowerDns/dnsdist: route incoming UDP/TCP DNS queries to a pool of DoH servers"
date: 2021-10-20T00:00:00+01:00
draft: false
tags: ['squid', 'proxy', 'docker']
---

This tutorial explained how to deploy the following [Squid 5](https://github.com/dmachard/squid-docker) docker image
and how to configure-it with a custom configuration (basic auth user).

## Setup user/password store

```
yum install httpd-tools

mkdir squid && cd squid
touch passwords
htpasswd -c passwords [USERNAME]
```
Replace [USERNAME] with your username. You will be prompted for entering the password. Enter and confirm it. 

## Deploy the Squid container

Download the squid configuration bellow on your host. Run the container.

```
docker run --name squid -d -p 3128:3128/tcp -v $PWD/squid.conf:/opt/squid/etc/squid.conf -v $PWD/passwords:/opt/squid/etc/passwords dmachard/squid:latest
```

## Test

You can run the curl on your host to test-it.

```
curl -x http://admin:password@127.0.0.1:3128/ https://www.example.com
```

## Basic configuration

```
acl Safe_ports port 443
http_access deny CONNECT !Safe_ports

auth_param basic program /opt/squid/libexec/basic_ncsa_auth /opt/squid/etc/passwords
auth_param basic realm Squid proxy-caching server
auth_param basic credentialsttl 24 hours
auth_param basic casesensitive off
acl authenticated proxy_auth REQUIRED

http_access allow authenticated
http_access deny all

http_port 3128
```