---
title: "Blacklist IP addresses with DNS UPDATE control and dynamic blocking duration"
date: 2023-08-17T00:00:00+01:00
draft: false
tags: ['powerdns', 'dnsdist', 'dns', 'security', 'blacklist', 'ip', 'update']
---

A DNSdist configuration example to blacklist IP addresses with DNS UPDATE control and dynamic blocking duration.
This example is volatible, dnsdist will restart with an empty blocklist of IP addresses.

## DNSdist configuration

In the following example, DNSdist will forward all incoming queries to `1.1.1.1` by default.
Before that, two actions are defined

- the first one to intercept all DNS update to `blockip.local.dev`.
- the second one to blacklist IP in limited time.

The full dnsdist.conf example:

> The latest version of the configuration can be downloaded from [github](https://github.com/dmachard/lua-dnsdist-config-examples/).

```lua
-- basic config with one backend for test purpose only
setLocal("0.0.0.0:53", {})
newServer({address = "1.1.1.1:53", pool="default"})

-- blacklist IP code part begin, this code requires dnsdist 1.8 minimum
-- Creates a Runtime-modifiable IP address sets: https://dnsdist.org/advanced/timedipsetrule.html?highlight=timedipsetrule#TimedIPSetRule
blacklistedIPs=TimedIPSetRule()

-- function to convert bytes IP to IPv4 string format
function convertToIPv4(ip_bytes)
    local ipv4_string = ""
    for i = 1, #ip_bytes do
        ipv4_string = ipv4_string .. string.byte(ip_bytes:sub(i, i))
        if i < #ip_bytes then
            ipv4_string = ipv4_string .. "."
        end
    end
    return ipv4_string
end

-- function to convert bytes IP to IPv6 string format
function convertToIPv6(ip_bytes)
    local ipv6_string = ""
    for i = 1, #ip_bytes, 2 do
        local hex_bytes = string.format("%02X%02X", ip_bytes:byte(i), ip_bytes:byte(i+1))
        ipv6_string = ipv6_string .. hex_bytes
        if i < #ip_bytes-1 then
            ipv6_string = ipv6_string .. ":"
        end
    end
    return ipv6_string
end

-- Parse the DNS UPDATE query to get IP addresses to block
function onRegisterIP(dq)
    local packet = dq:getContent()

    local overlay = newDNSPacketOverlay(packet)
    local countRecords = overlay:getRecordsCountInSection(DNSSection.Authority)
    
    if countRecords == 0 then
        errlog("blacklist error: invalid dns update")
        return DNSAction.ServFail, ""
    end

    for idx=0, countRecords-1 do
        local record = overlay:getRecord(idx)
        local ip_string = ""
        ip_bytes = string.sub(packet, record.contentOffset+1, record.contentOffset+record.contentLength)

        -- ip4 record
        if record.type == 1 then
            ip_string = convertToIPv4(ip_bytes)
        end
        -- ip6 record
        if record.type == 28 then
            ip_string = convertToIPv6(ip_bytes)
        end

        if ip_string == "0.0.0.0" and record.ttl == 0 then
            infolog("reset all blacklisted ips")
            blacklistedIPs:clear()
        else
            infolog("blacklisting IP: " .. ip_string .. " during " .. record.ttl .. " seconds")
        end
        blacklistedIPs:add(newCA(ip_string), record.ttl)
    end

    -- turn query in reply on success
    dq.dh:setQR(true)
    return DNSAction.HeaderModify, ""
end

-- register the IP address to blacklist with DNS UPDATE
-- to block IP during 60s: ./blockip_nsupdate.sh 172.17.0.1 60
-- to unblock all IPs: ./blockip_nsupdate.sh 0.0.0.0 0
addAction(AndRule({makeRule("blockip.local.dev"), OpcodeRule(DNSOpcode.Update)}), LuaAction(onRegisterIP))

-- Refused all IP addresses blacklisted
addAction(blacklistedIPs:slice(), RCodeAction(DNSRCode.REFUSED))

-- default rule
addAction( AllRule(), PoolAction("default"))
```

## How to block an IP address

To add IP addresses, you must send a DNS UPDATE to the domain `blockip.local.dev` with one or more ressource records.
The ressource must A or AAAA types. The TTL of RR will be used to set the blocking time in seconds.

Usage `nsupdate` command to block IP. In this example the IP `172.17.0.1` will be blocked during 60 seconds.

```bash
$ nsupdate --d -v
> server <dnsdist_ip> <dnsdist_port>
> zone blockip.local.dev
>
> update add blockip.local.dev 60 A 172.17.0.1
> send


$ sudo docker logs dnsdist
blacklisting IP: 172.17.0.1 during 60 seconds
```

To unblock all IP addresses, send a DNS update with zero TTL a record

```bash
> update add blockip.local.dev 0 A 0.0.0.0
> send

$ sudo docker logs dnsdist
reset all blacklisted ips
```

## Testing: make some DNS resolutions

Docker is used in this example, so the source IP is `172.17.0.1`, to block it for 60 seconds:

> The script `blockip_nsupdate.sh` is available in the following [repository](https://github.com/dmachard/lua-dnsdist-config-examples/).

```bash
$ ./blockip_nsupdate.sh 172.17.0.1 3600
Sending update to 127.0.0.1#5553
Outgoing update query:
;; ->>HEADER<<- opcode: UPDATE, status: NOERROR, id:  40574
;; flags:; ZONE: 1, PREREQ: 0, UPDATE: 1, ADDITIONAL: 0
;; ZONE SECTION:
;blockip.local.dev.             IN      SOA

;; UPDATE SECTION:
blockip.local.dev.      60      IN      A       172.17.0.1


Reply from update query:
;; ->>HEADER<<- opcode: UPDATE, status: NOERROR, id:  40574
;; flags: qr; ZONE: 1, PREREQ: 0, UPDATE: 1, ADDITIONAL: 0
;; ZONE SECTION:
;blockip.local.dev.             IN      SOA

;; UPDATE SECTION:
blockip.local.dev.      60      IN      A       172.17.0.1
```

Make a DNS resolution; the status of the response should be `REFUSED`.

```bash
$ dig @127.0.0.1 www.google.com 

; <<>> DiG 9.18.12-0ubuntu0.22.04.2-Ubuntu <<>> @127.0.0.1 www.google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 29967
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.google.com.                        IN      A
```

Retry after 60s, the resolution is OK.

```bash
$ dig @127.0.0.1 www.google.com +short
142.250.185.132
```
