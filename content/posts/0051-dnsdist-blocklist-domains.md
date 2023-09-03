---
title: "Temporarily block domains with DNS notify with DNSdist"
summary: "A DNSdist configuration example to block or unblock domains temporarily and in a dynamic way with DNS notify."
date: 2023-08-12T00:00:00+01:00
draft: false
tags: ['powerdns', 'dnsdist', 'dns', 'security', 'blocklist']
---

# Temporarily block domains with DNS notify with DNSdist

A DNSdist configuration example to block or unblock domains temporarily and in a dynamic way with DNS notify.
This example is volatible, a restart of the dnsdist will erase the blocklist. If you want to keep the blocklist even after restart, you can refer to the post [blackhole/spoofing domains with external files](https://github.com/dmachard/lua-dnsdist-config-examples/security_blackhole_domains.lua)

## DNSdist configuration

In the following example, DNSdist will forward all incoming queries to `1.1.1.1` by default.
Before that, an action is defined to check if the domain must be refused or not.

The full dnsdist.conf example:

> The latest version of the configuration can be downloaded from [github](https://github.com/dmachard/lua-dnsdist-config-examples/).

```lua
-- basic config with one backend for test purpose only
setLocal("0.0.0.0:53", {})
newServer({address = "1.1.1.1:53", pool="default"})

-- Creates a new SuffixMatchNode: https://dnsdist.org/reference/config.html?highlight=newsuffixmatchnode
local blackholeDomains = newSuffixMatchNode()

-- function to add or remove domain from the suffixMatchNode
local function onRegisterDomain(dq)
    if blackholeDomains:check(dq.qname) then
        infolog("removing domain: " ..  dq.qname:toString() .. " from blacklist")
        blackholeDomains:remove(dq.qname)
    else
        infolog("blacklisting domain: " ..  dq.qname:toString())
        blackholeDomains:add(dq.qname)
    end
    return DNSAction.Spoof, "success"
end

-- Check if the given qname is a sub-domain of one of those in the set
local function onBlacklistDomain(dq)
    if blackholeDomains:check(dq.qname) then
        return DNSAction.Refused
    else
        return DNSAction.None, ""      -- no action
    end
end

-- register domain to block or unblock from the DNS notify
addAction(OpcodeRule(DNSOpcode.Notify), LuaAction(onRegisterDomain))

-- Refused all domains blacklisted
addAction(AllRule(), LuaAction(onBlacklistDomain))

-- default rule
addAction( AllRule(), PoolAction("default"))
```

## How to block/unblock a domain

To add a domain in the blacklist, you must send a DNS notify to the domain (or a part of) to block.

To block the domain `fbcdn.net` and all subdomains, use the *dig* command to send a DNS notify.
You can also display the DNSdist logs to see log messages.

```bash
$ dig @127.0.0.1 +opcode=notify +tcp fbcdn.net +short
success.

$ sudo docker logs dnsdist
blacklisting domain: fbcdn.net.
```

To unblock the domain, send again the same DNS notify.

```bash
$ dig @127.0.0.1 +opcode=notify +tcp fbcdn.net +short
success.

$ sudo docker logs dnsdist
removing domain: fbcdn.net.
```

## Testing: make some DNS resolutions

Use the *dig* command on the domain `static.xx.fbcdn.net`
The status of the response is `REFUSED` because of the blacklist.

```bash
$ dig @127.0.0.1 static.xx.fbcdn.net

; <<>> DiG 9.18.12-0ubuntu0.22.04.2-Ubuntu <<>> @127.0.0.1 static.xx.fbcdn.net
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 51928
;; flags: qr rd ad; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;static.xx.fbcdn.net.           IN      A

;; Query time: 4 msec
;; SERVER: 127.0.0.1#5553(127.0.0.1) (TCP)
;; WHEN: Sat Aug 12 21:43:41 CEST 2023
;; MSG SIZE  rcvd: 48
```

Remove the domain `fbcdn.net` from the blocklist

```bash
$ dig @127.0.0.1 +opcode=notify +tcp fbcdn.net +short
success.

$ sudo docker logs dnsdist
removing domain: fbcdn.net. from blacklist
```

Try again to make the DNS resolution, this time the query is no more refused.

```bash
$ dig @127.0.0.1 static.xx.fbcdn.net +short
scontent.xx.fbcdn.net.
163.70.128.23
```
