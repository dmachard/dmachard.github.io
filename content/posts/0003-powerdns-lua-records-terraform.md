---
title: "PowerDNS/auth: how to manage LUA records with terraform"
date: 2021-05-22T00:00:00+01:00
draft: false
tags: ["dns", 'powerdns', 'terraform']
---

This post details how to manage LUA records with dynamic updates and [terraform](https://registry.terraform.io/providers/dmachard/powerdns-gslb/latest/docs) with your authoritative server.

## Requirements

Enable LUA records and DNS update to your pdns.conf

```
enable-lua-records=yes
dnsupdate=yes
```

Create a Tsig key and set metadata to your zone to authorize DNSUPDATE and AXFR with TSIG authentication.

```
TSIG-ALLOW-DNSUPDATE
TSIG-ALLOW-AXFR
```

## Provider configuration

The documentation of the provider *powerdns-gslb* is available in the terraform [registry](https://registry.terraform.io/providers/dmachard/powerdns-gslb/latest/docs)

1. Create a main.tf file

2. Install the provider "powerdns-glsb" then, run `terraform init`. 

```
terraform {
  required_providers {
    powerdns-gslb = {
      source = "dmachard/powerdns-gslb"
      version = "1.3.0"
    }
  }
}
```

3. Configure your provider with address of the DNS server to send updates to and TSIG authentication parameters
```
provider "powerdns-gslb" {
    server        = "10.0.0.210"
    key_name      = "test."
    key_algo      = "hmac-sha256"
    key_secret    = "SxEKov9vWTM+c7k9G6ho5nK.....n5nND5BOHzE6ybvy0+dw=="
}
```

## Create custom LUA record

Create the source `powerdns-gslb_lua` then, run `terraform apply`. 

```
resource "powerdns-gslb_lua" "svc1" {
  zone = "home.internal."
  name = "test_lua"
  record {
    rrtype = "A"
    ttl = 5
    snippet = "ifportup(8082, {'10.0.0.1', '10.0.0.2'})"
  }
}
```

You can removed the record by running `terraform destroy`. 

## Create pre-configured record

Some resources are available for ifurlup, ifportup, pickrandom and wpickrandom

```
resource "powerdns-gslb_pickrandom" "foo" {
  zone = "home.internal."
  name = "test_pickrandom"
  record {
    rrtype = "A"
    ttl = 5
    addresses = [ 
      "127.0.0.1",
      "127.0.0.2",
    ]
  }
}
```