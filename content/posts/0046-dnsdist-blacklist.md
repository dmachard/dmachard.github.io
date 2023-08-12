---
title: "PowerDNS dnsdist: dynamic blacklist with CDB database"
date: 2023-07-01T00:00:00+01:00
draft: false
tags: ['powerdns', 'dnsdist', 'blacklist', 'cdb']
---

A DNSdist configuration example to block big list of ads/malwares domains effectively
with a CDB database and dynamic reload.

Download the following CDB blocklist file https://github.com/dmachard/blocklist-domains
and put-it in /etc/dnsdist/conf.d/

The latest version of the configuration can be downloaded from [github](https://github.com/dmachard/lua-dnsdist-config-examples/).

```lua
--- open your CDB database 
--- dnsdist with reload this database every 3600s
kvs = newCDBKVStore("/etc/dnsdist/conf.d/blocklist.cdb", 3600)

-- block domains ?
addAction(KeyValueStoreLookupRule(kvs, KeyValueLookupKeyQName(false)), SetTagAction('policy_block', ''))
addAction(TagRule('policy_block'), SpoofAction({"127.0.0.1", "::1"}))

--- or answer with NXDOMAIN
--- addAction(TagRule('policy_block'), RCodeAction(DNSRCode.NXDOMAIN))
```

Official documentation links

- newCDBKVStore https://dnsdist.org/reference/kvs.html#newCDBKVStore
- KeyValueStoreLookupRule https://dnsdist.org/rules-actions.html#KeyValueStoreLookupRule
- KeyValueLookupKeyQName https://dnsdist.org/reference/kvs.html#KeyValueLookupKeyQName
