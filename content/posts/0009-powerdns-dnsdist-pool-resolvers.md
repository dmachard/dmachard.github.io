---
title: "PowerDns/dnsdist: how to send all your DNS queries to a pool of public resolvers"
date: 2021-09-28T00:00:00+01:00
draft: false
tags: ['dns', 'powerdns', 'dnsdist']
---

In this very basic example, the goal is to send all your local DNS queries (udp/tcp) to a pool of public resolvers (without encryption).
[dnsdist](https://dnsdist.org/) is configured to make a load balancing (round robin) between all public resolvers configured.
A dns cache is enabled to optimize the traffic. We assume you have dnsdist 1.6 minimum installed on your machine.

Configuation: /etc/dnsdist/dnsdist.conf

```lua
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

-- members definition
newServer({
  name = "google",
  address = "8.8.8.8:53",
  pool = pool_resolv,
})

newServer({
  name = "quad9",
  address = "9.9.9.9:53",
  pool = pool_resolv,
})

-- set the load balacing policy to use
setPoolServerPolicy(roundrobin, pool_resolv)

-- enable cache for the pool
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