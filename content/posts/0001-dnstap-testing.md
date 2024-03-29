---
title: "Enabling DNStap logging on most popular DNS servers"
summary: "This post details how to enable the dnstap feature in most popular dns server software."
date: 2020-05-23T00:00:00+01:00
draft: false
tags: ["dns", 'dnstap', 'logs']
pin: true
---

# Enabling DNStap logging on most popular DNS servers

This post details how to enable the dnstap feature in main open source dns servers.

[dnstap](https://dnstap.info/) is a flexible, structured binary log format for DNS servers.
It uses [Protocol Buffers](https://protobuf.dev/) to encode DNS packets in events.

dnstap can encode any DNS messages with network informations like ip and port. It includes client queries and responses.

## Introduction

This [dnstap](https://dnstap.info/) feature has been tested with success with the following dns servers:

|                                                        | RQ/RS   | CQ/CR | AQ/AR | FQ/FR | Unix Socket | TCP Stream | TLS support | Extra Field |
|--------------------------------------------------------|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|
| [Bind](https://kb.isc.org/docs/aa-01342)               |   x   |   x   |   x   |       |      x      |      x     |             |             |
| [PowerDNS recursor](https://docs.powerdns.com/recursor/lua-config/protobuf.html)   |   x   |     |    |     |    x      |    x     |       |       |
| [PowerDNS dnsdist](https://dnsdist.org/reference/dnstap.html)     |       |   x   |       |       |      x      |      x     |             |      x      |
| [NSD](#nsd)       |       |       |   x   |       |      x      |            |             |             |
| [Unbound](https://nlnetlabs.nl/documentation/unbound/unbound.conf/)   |   x   |   x   |       |   x   |      x      |      x     |      x      |             |
| [CoreDNS](https://coredns.io/plugins/dnstap/)              |       |   x   |       |   x   |      x      |      x     |      x      |      x      |
| [Knot Resolver](https://knot-resolver.readthedocs.io/en/stable/modules-dnstap.html) |       |   x   |       |       |      x      |            |             |             |
| [Knot DNS](https://www.knot-dns.cz/docs/2.6/html/modules.html#dnstap-dnstap-traffic-logging)      |       |       |   X   |       |      x      |            |             |             |

## ISC bind

![bind 9.11.22](https://img.shields.io/badge/9.11.22-tested-green) ![bind 9.16.10](https://img.shields.io/badge/9.16.10-tested-green)

Dnstap messages supported:

* RESOLVER_QUERY
* RESOLVER_RESPONSE
* CLIENT_QUERY
* CLIENT_RESPONSE
* AUTH_QUERY
* AUTH_RESPONSE

### Build with dnstap support

Since 9.16 version, the dnstap feature is enabled before that you need to download latest source and build-it with dnstap support:

```bash
./configure --enable-dnstap
make && make install
```

### Unix socket

Update the configuration file `/etc/named.conf` to activate the dnstap feature:

```bash
options {
    dnstap { client; auth; resolver; forwarder; };
    dnstap-output unix "/var/run/named/dnstap.sock";
    dnstap-identity "dns-bind";
    dnstap-version "bind";
}
```

Execute the dnstap receiver with `named` user:

```bash
su - named -s /bin/bash -c "dnstap_receiver -u "/var/run/named/dnstap.sock""
```

If you have some troubles take a look to [selinux](https://gitlab.isc.org/isc-projects/bind9/-/issues/2356#note_185516)

### TCP stream

Not supported on Bind! You can apply the following workaround with the `socat` or `stunnel` command.

```bash
while true; do socat unix-listen:/var/run/dnsdist/dnstap.sock tcp4-connect:<ip_dnstap_receiver>:<port_dnstap_receiver>,forever,interval=10, fork; sleep 1; done
```

## PowerDNS - pdns-recursor

![pdns-recursor 4.3.4](https://img.shields.io/badge/4.3.4-tested-green) ![pdns-recursor 4.4.0](https://img.shields.io/badge/4.4.0-tested-green)

Dnstap messages supported:

* RESOLVER_QUERY
* RESOLVER_RESPONSE

### Unix socket

Update the configuration file to activate the dnstap feature:

```bash
vim /etc/pdns-recursor/recursor.conf
lua-config-file=/etc/pdns-recursor/recursor.lua

vim /etc/pdns-recursor/recursor.lua
dnstapFrameStreamServer("/var/run/pdns-recursor/dnstap.sock")
```

Execute the dnstap receiver with `pdns-recursor` user:

```bash
su - pdns-recursor -s /bin/bash -c "dnstap_receiver -u "/var/run/pdns-recursor/dnstap.sock""
```

### TCP stream

Update the configuration file to activate the dnstap feature with tcp mode
and execute the dnstap receiver in listening tcp socket mode:

```bash
vim /etc/pdns-recursor/recursor.conf
lua-config-file=/etc/pdns-recursor/recursor.lua

vim /etc/pdns-recursor/recursor.lua
dnstapFrameStreamServer("10.0.0.100:6000")
```

Note: TCP stream are only supported with a recent version of libfstrm.

## PowerDNS - dnsdist

![dnsdist 1.8](https://img.shields.io/badge/1.8-tested-green) ![dnsdist 1.7](https://img.shields.io/badge/1.7-tested-green) ![dnsdist 1.6](https://img.shields.io/badge/1.6-tested-green) ![dnsdist 1.5](https://img.shields.io/badge/1.5-tested-green) ![dnsdist 1.4](https://img.shields.io/badge/1.4.0-tested-green)

Dnstap messages supported:

* CLIENT_QUERY
* CLIENT_RESPONSE

### Unix socket

Create the dnsdist folder where the unix socket will be created:

```bash
mkdir -p /var/run/dnsdist/
chown dnsdist.dnsdist /var/run/dnsdist/
```

Update the configuration file `/etc/dnsdist/dnsdist.conf` to activate the dnstap feature:

```bash
fsul = newFrameStreamUnixLogger("/var/run/dnsdist/dnstap.sock")
addAction(AllRule(), DnstapLogAction("dnsdist", fsul))
addResponseAction(AllRule(), DnstapLogResponseAction("dnsdist", fsul))
-- Cache Hits
addCacheHitResponseAction(AllRule(), DnstapLogResponseAction("dnsdist", fsul))
```

Execute the dnstap receiver with `dnsdist` user:

```bash
su - dnsdist -s /bin/bash -c "dnstap_receiver -u "/var/run/dnsdist/dnstap.sock""
```

### TCP stream

Update the configuration file `/etc/dnsdist/dnsdist.conf` to activate the dnstap feature
with tcp stream and execute the dnstap receiver in listening tcp socket mode:

```bash
fsul = newFrameStreamTcpLogger("127.0.0.1:8888")
addAction(AllRule(), DnstapLogAction("dnsdist", fsul))
addResponseAction(AllRule(), DnstapLogResponseAction("dnsdist", fsul))
-- Cache Hits
addCacheHitResponseAction(AllRule(), DnstapLogResponseAction("dnsdist", fsul))
```

## NLnetLabs - nsd

![nsd 4.3.2](https://img.shields.io/badge/4.3.2-tested-green)

Dnstap messages supported:

* AUTH_QUERY
* AUTH_RESPONSE

### Build with dnstap support

Download latest source and build-it with dnstap support:

```bash
./configure --enable-dnstap
make && make install
```

### Unix socket

Update the configuration file `/etc/nsd/nsd.conf` to activate the dnstap feature:

```yaml
dnstap:
    dnstap-enable: yes
    dnstap-socket-path: "/var/run/nsd/dnstap.sock"
    dnstap-send-identity: yes
    dnstap-send-version: yes
    dnstap-log-auth-query-messages: yes
    dnstap-log-auth-response-messages: yes
```

Execute the dnstap receiver with `nsd` user:

```bash
su - nsd -s /bin/bash -c "dnstap_receiver -u "/var/run/nsd/dnstap.sock""
```

## NLnetLabs - unbound

![unbound 1.11.0](https://img.shields.io/badge/1.11.0-tested-green) ![unbound 1.12.0](https://img.shields.io/badge/1.12.0-tested-green) ![unbound 1.13.0](https://img.shields.io/badge/1.13.0-tested-green)

Dnstap messages supported:

* CLIENT_QUERY
* CLIENT_RESPONSE
* RESOLVER_QUERY
* RESOLVER_RESPONSE

### Build with dnstap support

Download latest source and build-it with dnstap support:

```bash
./configure --enable-dnstap
make && make install
```

### Unix socket

Update the configuration file `/etc/unbound/unbound.conf` to activate the dnstap feature:

```yaml
dnstap:
    dnstap-enable: yes
    dnstap-socket-path: "dnstap.sock"
    dnstap-send-identity: yes
    dnstap-send-version: yes
    dnstap-log-resolver-query-messages: yes
    dnstap-log-resolver-response-messages: yes
    dnstap-log-client-query-messages: yes
    dnstap-log-client-response-messages: yes
    dnstap-log-forwarder-query-messages: yes
    dnstap-log-forwarder-response-messages: yes
```

Execute the dnstap receiver with `unbound` user:

```bash
su - unbound -s /bin/bash -c "dnstap_receiver -u "/usr/local/etc/unbound/dnstap.sock""
```

### TCP stream

Update the configuration file `/etc/unbound/unbound.conf` to activate the dnstap feature
with tcp mode and execute the dnstap receiver in listening tcp socket mode:

```yaml
dnstap:
    dnstap-enable: yes
    dnstap-socket-path: ""
    dnstap-ip: "10.0.0.100@6000"
    dnstap-tls: no
    dnstap-send-identity: yes
    dnstap-send-version: yes
    dnstap-log-client-query-messages: yes
    dnstap-log-client-response-messages: yes
```

### TLS stream

Update the configuration file `/etc/unbound/unbound.conf` to activate the dnstap feature
with tls mode and execute the dnstap receiver in listening tcp/tls socket mode:

```yaml
dnstap:
    dnstap-enable: yes
    dnstap-socket-path: ""
    dnstap-ip: "10.0.0.100@6000"
    dnstap-tls: yes
    dnstap-send-identity: yes
    dnstap-send-version: yes
    dnstap-log-client-query-messages: yes
    dnstap-log-client-response-messages: yes
```

## CoreDNS

![coredns 1.11.1](https://img.shields.io/badge/1.11.1-tested-green) ![coredns 1.8.4](https://img.shields.io/badge/1.8.4-tested-green) ![coredns 1.8.0](https://img.shields.io/badge/1.8.0-tested-green)

Dnstap messages supported:

* CLIENT_QUERY
* CLIENT_RESPONSE
* FORWARDER_QUERY
* FORWARDER_RESPONSE

### Unix socket

corefile example

```bash
.:53 {
    dnstap /tmp/dnstap.sock full
    forward . 8.8.8.8:53
}
```

Then execute CoreDNS with your corefile

```bash
 ./coredns -conf corefile
```

### TCP stream

corefile example

```bash
.:53 {
        dnstap tcp://10.0.0.51:6000 full
        forward . 8.8.8.8:53
}
```

Then execute CoreDNS with your corefile

```bash
 ./coredns -conf corefile
```

### TLS stream

corefile example

```bash
.:53 {
        dnstap tls://10.0.0.51:6000 full {
          skip-verify
        }
        forward . 8.8.8.8:53
}
```

Then execute CoreDNS with your corefile

```bash
 ./coredns -conf corefile
```

## CZ-NIC - Knot Resolver

### Unix socket

corefile example

```bash
net.listen("0.0.0.0", 5553)

modules.load('nsid')
nsid.name('instance1')

modules = {
    dnstap = {
        socket_path = "/tmp/dnstap.sock",
        identity =  nsid.name() or "",
        version = "knot-resolver" .. package_version(),
        client = {
            log_queries = true,
            log_responses = true,
        },
    }
}
```

Then execute the Knot Resolver, example with docker

```bash
sudo docker run -d -v $PWD/kresd.conf:/etc/knot-resolver/kresd.conf --name=knot --network=host cznic/knot-resolver -n -c /etc/knot-resolver/kresd.conf
```
