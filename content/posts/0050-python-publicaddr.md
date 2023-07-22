---
title: "Python Module to get your V4/V6 public IP from random providers in several ways (DNS, HTTPS or STUN)"
date: 2023-07-22T01:00:00+01:00
draft: false
tags: ['python', 'publicip', 'ipv4', 'ipv6', 'package']
---

Simple python module for getting your public IP V4 and V6 from several providers in random mode with also several protocols (DNS, HTTPS and STUN).

Source code available in [github](https://github.com/dmachard/python-publicaddr/tree/main) under MIT licence.

The following providers are supported:
- Google (DNS & HTTP & STUN)
- Cloudflare (DNS & HTTP)
- OpenDNS (DNS)
- Akamai (DNS & HTTP)
- Ipify (HTTP)
- Icanhazip (HTTP)
- Matrix (STUN)
- Framasoft (STUN)

# Installation

This module can be installed from [pypi](https://pypi.org/project/publicaddr/) website

```bash
pip install publicaddr
```

# Lookup for IPv4 and v6

Lookup for your public IPs from random providers with DNS or HTTP protocols with 3 retries if no ips are returned.
This is the default behaviour of the `lookup` function.

```python
import publicaddr

publicaddr.lookup()
{'ip4': 'x.x.x.x', 'ip6': 'x:x:x:x:x:x:x:x', 'provider': 'opendns',
'proto': 'dns', 'duration': '0.037'}
```

# Lookup for public IP with specific protocol

Lookup for your public IPs from random DNS providers with specific protocol.

```python
import publicaddr

publicaddr.lookup(providers=publicaddr.DNS, retries=2)
{'ip4': 'x.x.x.x', 'ip6': 'x:x:x:x:x:x:x:x', 'provider': 'opendns',
'proto': 'dns', 'duration': '0.037'}
```

Default constants for transport protocol:
- `publicaddr.HTTPS`
- `publicaddr.DNS`
- `publicaddr.STUN`

# Get IPv4 or IPv6 only

Get your public IPv4 with default provider (Google with DNS protocol).

```python
import publicaddr

publicaddr.get(ip=publicaddr.IPv4)
{'ip': 'x.x.x.x', 'duration': '0.025'}
```

Default constants for IP version:
- `publicaddr.IPv4`
- `publicaddr.IPv6`

# Get IP with specific provider

Example to use the provider Cloudflare instead of the default one.

```python
import publicaddr

myip = publicaddr.get(provider=publicaddr.CLOUDFLARE, proto=publicaddr.DNS)
{'ip': 'x:x:x:x:x:x:x:x', 'duration': '0.020'}
```

Default constants for providers:
- `publicaddr.CLOUDFLARE`
- `publicaddr.GOOGLE`
- `publicaddr.OPENDNS`
- `publicaddr.AKAMAI`
- `publicaddr.IPIFY`
- `publicaddr.ICANHAZIP`
- `publicaddr.MATRIX`
- `publicaddr.FRAMASOFT`