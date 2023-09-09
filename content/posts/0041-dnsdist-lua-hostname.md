---
title: "Custom LUA config with DNSdist"
summary: "This post gives some examples of LUA code to modify the behavior of your dnsdist"
date: 2021-11-21T00:00:00+01:00
draft: false
tags: ['powerdns', 'dnsdist']
---

# Custom LUA config with DNSdist

This post gives some examples of LUA code to modify the behavior of your dnsdist 
## Resolve hostname

This exemple enable to make dns resolution at startup.

```lua
local f = assert(io.popen('getent hosts <YOURDNSNAME> | cut -d " " -f 1', 'r'))
local dnscollector = f:read('*a') or "127.0.0.1"
f:close()
dnscollector = string.gsub(dnscollector, "\n$", "")

fstl = newFrameStreamTcpLogger(dnscollector.. ":6000")
```

## Get hostname

Example to get the hostname of your machine and reuse-it in your dnsdist config

```lua
local f = io.popen ("/bin/hostname")
local hostname = f:read("*a") or "dnsdist"
f:close()
hostname = string.gsub(hostname, "\n$", "")

addAction(AllRule(), DnstapLogAction(hostname, fstl))
addResponseAction(AllRule(), DnstapLogResponseAction(hostname, fstl))
addCacheHitResponseAction(AllRule(), DnstapLogResponseAction(hostname, fstl))
```