---
title: "PowerDns/dnsdist: route incoming UDP/TCP DNS queries to a pool of DoH servers"
date: 2021-10-20T00:00:00+01:00
draft: false
tags: ['dns', 'powerdns', 'dnsdist', 'doh']
---

The goal of this tutorial is to send all your local DNS queries (udp/tcp) to a pool of DoH public resolvers with [dnsdist](https://dnsdist.org/) load balancer. A dns cache is also enabled to optimize the traffic. We also assume you have a docker environement to follow this tutorial.

# Table of contents

* [Prepare configuration](#prepare-configuration)
* [Deploy dnsdist container](#deploy-dnsdist-container)
* [Test DNS resolution](#test-dns-resolution)
* [Jinja configuration template](#jinja-configuration-template)

## Prepare configuration

Before to start, install some useful python tools to prepare the configuration

```bash
pip install dnsdist_console j2cli
```

Now, we will create the configuration from the jinja template *dnsdist-doh.j2* provided below.
Download the template on your host before to continue.

The following variables need to be exported:
- DNSDIST_PWD
- DNSDIST_POOL

Update the list of resolvers and the password according to your needs:

```bash
export DNSDIST_PWD="superpassword" \
export DNSDIST_POOL='{"resolv": [{"addr": "8.8.8.8:443", "name": "dns.google"},{"addr": "1.1.1.1:443", "name": "cloudflare-dns.com"}]}'
```

```bash
export DNSDIST_API_KEY=$(python3 -c "from dnsdist_console import HashPassword as H;print(H().generate(\"$(echo $DNSDIST_PWD)\"))") \
export DNSDIST_WEB_KEY=$DNSDIST_API_KEY \
export DNSDIST_CONSOLE_KEY=$(python3 -c "from dnsdist_console import Key;print(Key().generate())")
```

Use the j2 command to generate the custom configuration

```bash
echo $DNSDIST_POOL | j2 --format=json dnsdist-doh.j2 > dnsdist-doh.lua
```

The configuration template is just provided for example, your can updated-it according to your needs.
This config can be used to send all your local DNS queries to a pool of public resolvers.

## Deploy dnsdist container

Deploy the dnsdist image with the custom DoH configuration

```bash
docker run -d -p 53:53/udp -p 53:53/tcp -p 8083:8083 --restart unless-stopped --name=dnsdist \
--volume=$PWD/dnsdist-doh.lua:/etc/dnsdist/conf.d/dnsdist.conf:ro powerdns/dnsdist-17:1.7.0-alpha2
```

Check if the container is running properly

```bash
 docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS         PORTS                                                                      NAMES
8ff7d92ddd1d   powerdns/dnsdist-17:1.7.0-alpha2     "/usr/bin/tini -- /u…"   2 seconds ago   Up 2 seconds   0.0.0.0:53->53/tcp, 0.0.0.0:8083->8083/tcp, 0.0.0.0:53->53/udp, 5199/tcp   dnsdist
```

## Test DNS resolution

```bash
# dig @::1 www.google.fr +tcp +short
216.58.215.35

# dig @127.0.0.1 www.google.fr +short
216.58.215.35
```

## Jinja configuration template

```lua
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
caStore = "/etc/dnsdist/conf.d/tls-ca-bundle.pem"

-- members definitions
{% for res in resolv %}
newServer({
  address = "{{ res["addr"] }}",
  tls = "openssl",
  dohPath = "/dns-query",
  caStore = caStore,
  subjectName = "{{ res["name"] }}",
  validateCertificates = false,
  pool = pool_resolv,
})

{% endfor %}

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