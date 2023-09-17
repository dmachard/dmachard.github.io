---
title: "Configuration examples for DNSdist PowerDNS"
summary: "This post aims to share different configurations for DNSdist"
date: 2023-09-17T00:00:00+01:00
draft: false
tags: ['powerdns', 'dnsdist']
pin: true
---

# Configuration examples for DNSdist PowerDNS

This post aims to share different configurations for [DNSdist](https://dnsdist.org/).
Each examples can be tested with the following [docker guide](https://github.com/dmachard/lua-dnsdist-config-examples#run-config-from-docker).
Before to start take a look to the [default](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/default_config.lua) one provided by PowerDNS.

## Administration

- [How to enable web admin and console interfaces](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/admin_config.lua)

## Routing DNS traffic

- [Match Qname with regular expression](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/routing_regex.lua)
- [Tag your traffic and applied specified rules on it](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/routing_tag_traffic.lua)

## DNS Security

- [Ads/Malwares blocking with external CDB database](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/security_blacklist_cdb.lua)
- [DNS tunneling blocking](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/security_blocking_dnstunneling.lua)
- [Blackhole/spoofing domains with external files](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/security_blackhole_domains.lua)
- [Blacklist IP addresses with DNS UPDATE control and dynamic blocking duration](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/security_blacklist_ip_dnsupdate.lua)
- [Blacklist IP during XX seconds, the list of IPs is managed with DNS notify and TTL for duration](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/security_blacklist_ip_notify.lua)
- [List of temporarily blocked domains, the list is managed with DNS notify](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/security_blocklist_domains.lua)
- [Spoofing DNS responses like TXT, A, AAAA, MX and more...](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/security_spoofing_qtype.lua)

## Logging DNS traffic

- [Remote DNS logging with DNSTAP protocol](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/logging_dnstap.lua)
- [Remote DNS logging with Protobuf protocol](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/logging_protobuf.lua)

## Miscs

- [Full configuration with load balancing on public DNS resolvers](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/miscs_basic_config.lua)
- [Flush cache for domain with DNS NOTIFY](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/miscs_cache_flush_notify.lua)
- [Echo capability of ip address from domain name for development](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/miscs_echoip.lua)
- [Resolve hostname from config](https://github.com/dmachard/lua-dnsdist-config-examples/blob/main/miscs_resolve_hostname.lua)
