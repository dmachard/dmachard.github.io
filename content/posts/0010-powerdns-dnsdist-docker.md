---
title: "PowerDns/dnsdist: how to run dnsdist in a Docker Container with custom configuration file"
date: 2021-10-01T00:00:00+01:00
draft: false
tags: ['dns', 'powerdns', 'dnsdist', 'docker']
---

The dnsdist product is available in the official [dockerhub registry](https://hub.docker.com/u/powerdns) of PowerDNS.
This post details how to execute [dnsdist](https://dnsdist.org/) in **docker container**, **custom configuration file** and **environnement variables**. We assume you have a containers environnement already available.

## Custom configuration

Before to start, install some useful python tools to prepare the configuration

```
pip install dnsdist_console j2cli
```

Now, we will create the configuration from the jinja template *dnsdist.lua* provided below.
To do that, you need to export some environment variables.

The following variables need to be exported:
- DNSDIST_CONSOLE_KEY
- DNSDIST_API_KEY
- DNSDIST_WEB_KEY
- DNSDIST_RESOLVERS

Copy/Paste and update the list of resolvers according to your needs.

```bash
export DNSDIST_RESOLVERS='{"resolvers": ["8.8.8.8","9.9.9.9"]}' \
export DNSDIST_CONSOLE_KEY=$(python3 -c "from dnsdist_console import Key;print(Key().generate())") \
export DNSDIST_API_KEY=$(python3 -c "import secrets; print(secrets.token_urlsafe(16))") \
export DNSDIST_WEB_KEY=$(python3 -c "import secrets; print(secrets.token_urlsafe(16))")
```

Finally use the j2 command to generate the custom configuration

```
echo $DNSDIST_RESOLVERS | j2 --format=json dnsdist.j2 > dnsdist.lua
```

The configuration template is just provided for example, your can updated-it according to your needs.
This config can be used to send all your local DNS queries to a pool of public resolvers.

## Deploy dnsdist container

Deploy the dnsdist image with the custom configuration

```bash
docker run -d -p 53:53/udp -p 53:53/tcp -p 8083:8083 --restart unless-stopped --name=dnsdist01 \
--volume=$PWD/dnsdist.lua:/etc/dnsdist/conf.d/dnsdist.conf:ro powerdns/dnsdist-16
```

Check if the container is running properly

```bash
 docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS         PORTS                                                                      NAMES
8ff7d92ddd1d   powerdns/dnsdist-16     "/usr/bin/tini -- /u…"   2 seconds ago   Up 2 seconds   0.0.0.0:53->53/tcp, 0.0.0.0:8083->8083/tcp, 0.0.0.0:53->53/udp, 5199/tcp   dnsdist01
```

## Connect to the console

Try to connect to the console and display the running configuration

```bash
# docker exec -it dnsdist01 dnsdist -c

> showPools()
Name                                    Cache         ServerPolicy Servers
                                                  leastOutstanding 
resolvers                             0/10000           roundrobin 8.8.8.8:53 8.8.8.8:53, 9.9.9.9:53 9.9.9.9:53

> showServers()
#   Name                 Address                       State     Qps    Qlim Ord Wt    Queries   Drops Drate   Lat Outstanding Pools
0   8.8.8.8:53           8.8.8.8:53                       up     0.0       0   1  1          0       0   0.0   0.0           0 resolvers
1   9.9.9.9:53           9.9.9.9:53                       up     0.0       0   1  1          0       0   0.0   0.0           0 resolvers
All      

> showRules()
#   Name                             Matches Rule                                                     Action
0                                          0 All                                                      to pool resolvers
```

## Test DNS resolution

```
# dig @::1 www.google.fr +tcp +short
216.58.215.35

# dig @127.0.0.1 www.google.fr +tcp +short
216.58.215.35
```

## Jinja configurationg template

```
---------------------------------------------------
-- Administration
---------------------------------------------------

-- cli
setKey("{{ env("DNSDIST_CONSOLE_KEY") }}")
controlSocket("127.0.0.1:5199")
addConsoleACL("127.0.0.1/8, ::1/128")

-- api
webserver("0.0.0.0:8083")
setWebserverConfig({
  apiKey = "{{ env("DNSDIST_API_KEY") }}",
  password = "{{ env("DNSDIST_WEB_KEY") }}",
  acl = "192.168.0.0/16",
})

---------------------------------------------------
-- Dns services
---------------------------------------------------

-- udp/tcp dns listening
setLocal("0.0.0.0:53", {})

-- dns caching
pc = newPacketCache(10000, {})

---------------------------------------------------
-- Pools
---------------------------------------------------

pool_resolv = "resolvers"

-- members definitions
{% for res in resolvers -%}
newServer({
  address = "{{ res }}",
  pool = pool_resolv,
})

{% endfor -%}

-- set the load balacing policy to use
setPoolServerPolicy(roundrobin, pool_resolv)

-- enable cache
getPool(pool_resolv):setCache(pc)


---------------------------------------------------
-- Rules
---------------------------------------------------

-- matches all incoming traffic and send-it to the pool of resolvers
addAction(
  AllRule(),
  PoolAction(pool_resolv)
)
```