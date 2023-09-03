---
title: "Echo capability of ip address from domain name for development with DNSdist"
summary: "A DNSdist configuration example to map any IP address to a hostname for development purpose."
date: 2023-07-22T00:00:00+01:00
draft: false
tags: ['powerdns', 'dnsdist', 'development', 'dns']
---

# Echo capability of ip address from domain name for development with DNSdist

A DNSdist configuration example to map any IP address to a hostname for development purpose.
The following formats are supported:

- 10.0.0.1.local.dev will return A record as A 10.0.0.1
- 192-168-1-250.local.dev will return A record as A 192.168.1.250
- app.10.8.0.1.local.dev will return A record as A 10.8.0.1
- app-116-203-255-68.local.dev will return A record as A 116.203.255.68
- customer1.app.10.0.0.1.local.dev will return A record as A 10.0.0.1
- customer-app1-127-0-0-1.local.dev will return A record as A 127.0.0.1
- 0a000803.local.dev will return A record as A 10.0.8.3
- app-c0a801fc.local.dev will return A record as 192.168.1.252
- customer3-app-7f000101.local.dev will return A record as 127.0.1.1

Invalid IP will return CNAME record.

In the following example, the `local.dev` domain is used to map IP addresses otherwise all other DNS queries are forwarded 
to the default backend `1.1.1.1`.

The full dnsdist.conf example:

> The latest version of the configuration can be downloaded from [github](https://github.com/dmachard/lua-dnsdist-config-examples/).

```lua
setLocal("0.0.0.0:53", {})
newServer({address = "1.1.1.1:53"})


-- Echo capability of ip address from domain name for development
local function onEchoIpAddress(dq)
        -- list of regexs to find ip in qname
        ipsregex = {}

        -- 10.0.0.1.local.dev maps to A 10.0.0.1
        -- 192-168-1-250.local.dev maps to A 192.168.1.250
        ipsregex.ip4  = "^(%d+[%.-]%d+[%.-]%d+[%.-]%d+)%."

        -- app.10.8.0.1.local.dev maps to A 10.8.0.1
        -- app-116-203-255-68.local.dev maps to A 116.203.255.68
        -- customer1.app.10.0.0.1.local.dev maps to A 10.0.0.1
        -- customer-app1-127-0-0-1.local.dev maps to A 127.0.0.1
        ipsregex.ip4name = "[^a-z0-9A-Z]%.?(%d+[%.-]%d+[%.-]%d+[%.-]%d+)%."

        -- 0a000803.local.dev maps to A 10.0.8.3
        ipsregex.ip4hex = "^([0-9a-fA-F][0-9a-fA-F][0-9a-fA-F][0-9a-fA-F][0-9a-fA-F][0-9a-fA-F][0-9a-fA-F][0-9a-fA-F])%."

        -- app-c0a801fc.local.dev maps to A 192.168.1.252
        -- customer3-app-7f000101.local.dev maps to 127.0.1.1
        ipsregex.ip4hexname = "[^a-z0-9A-Z]%.?([0-9a-fA-F][0-9a-fA-F][0-9a-fA-F][0-9a-fA-F][0-9a-fA-F][0-9a-fA-F][0-9a-fA-F][0-9a-fA-F])%."

        -- default ip to return on no matched
        local retip = "127.0.0.2"

        -- iter over each regex to find ip
        for k, pattern in pairs(ipsregex) do
           addr  = string.match(dq.qname:toString(), pattern)
           if addr then
             retip = string.gsub(addr, "-", ".")

             if k == "ip4hex" or k == "ip4hexname" then
                retip = tonumber(retip, 16)
             end

             break
           end
        end

        return DNSAction.Spoof, retip
end

-- rule to match local domain
addAction("local.dev.", LuaAction(onEchoIpAddress))

-- default rule
addAction( AllRule(), PoolAction("default"))
```

Use the `dig` command to test this configuration:

```bash
dig 10.0.0.1.local.dev

;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24739
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;10.0.0.1.local.dev.            IN      A

;; ANSWER SECTION:
10.0.0.1.local.dev.     60      IN      A       10.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#5553(127.0.0.1) (TCP)
;; WHEN: Sat Aug 12 21:16:14 CEST 2023
;; MSG SIZE  rcvd: 63
```
