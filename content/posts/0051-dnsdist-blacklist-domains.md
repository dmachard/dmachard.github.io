---
title: "PowerDNS DNSdist: block or unblock domain in dynamic way (DNS notify)"
date: 2023-08-12T00:00:00+01:00
draft: false
tags: ['powerdns', 'dnsdist', 'dns', 'security', 'blacklist']
---

A DNSdist configuration example to block or unblock domains in a dynamic way with DNS notify.

## DNSdist configuration

In the following example, DNSdist will forward all incoming queries to `1.1.1.1` by default.
Before that, an action is defined to check if the domain must be refused or not.

The full dnsdist.conf example:

```lua
setLocal("0.0.0.0:53", {})
newServer({address = "1.1.1.1:53", pool="default"})

local blackholeDomains = newSuffixMatchNode()

local function onRegisterDomain(dq)
    if blackholeDomains:check(dq.qname) then
        infolog("removing domain " ..  dq.qname:toString() .. " from blacklist")
        blackholeDomains:remove(dq.qname)
    else
        infolog("blacklisting the domain: " ..  dq.qname:toString())
        blackholeDomains:add(dq.qname)
    end
    return DNSAction.Spoof, "success"
end

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

> The latest version of the configuration can be downloaded from [github](https://github.com/dmachard/lua-dnsdist-config-examples/).

## Add a domain to the blocklist

To add a domain in the blacklist, you must send a DNS notify to the domain (or a part of) to block.

Example to block fbcdn.net and all subdomains:

```bash
dig @127.0.0.1 +opcode=notify +tcp fbcdn.net +short
success.
```

Display DNSdist logs to check if the blacklist is added

```bash
$ sudo docker logs dnsdist
blacklisting the domain: fbcdn.net.
```

> To unblock the domain, send again the same DNS notify.

## Test

Test the domain `static.xx.fbcdn.net`, it should be `REFUSED`

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

Remove `fbcdn.net` from the blocklist

```bash
$ dig @127.0.0.1 +opcode=notify +tcp fbcdn.net +short
success.

$ sudo docker logs dnsdist
removing domain fbcdn.net. from blacklist
```

Try again to make the DNS resolution, this time the query is no more refused.

```bash
$ dig @127.0.0.1 static.xx.fbcdn.net +short
scontent.xx.fbcdn.net.
163.70.128.23
```