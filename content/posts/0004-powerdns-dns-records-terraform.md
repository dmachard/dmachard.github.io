---
title: "PowerDNS/auth: how to manage DNS records with dynamic updates and terraform"
date: 2021-05-22T00:00:00+01:00
draft: false
tags: ["dns", 'powerdns', 'terraform']
---
# PowerDNS - How to use Terraform with DNS update

This post details how to manage DNS records with dynamic updates and [terraform](https://registry.terraform.io/providers/hashicorp/dns/latest/docs) with your authoritative server.

## Requirements

Enable DNS update to your pdns.conf

```
dnsupdate=yes
```

Create a Tsig key and set metadata to your zone to authorize DNSUPDATE and AXFR with TSIG authentication.

```
TSIG-ALLOW-DNSUPDATE
TSIG-ALLOW-AXFR
```

## Provider configuration

1. Create a main.tf file

2. Install the provider "dns" then, run `terraform init`. 

```
terraform {
  required_providers {
    dns = {
      source = "hashicorp/dns"
      version = "3.1.0"
    }
  }
}
```

3. Configure your provider with address of the DNS server to send updates to and TSIG authentication parameters

```
provider "dns" {
  update {
    server        = "192.168.0.1"
    key_name      = "example.com."
    key_algorithm = "hmac-md5"
    key_secret    = "3VwZXJzZWNyZXQ="
  }
}
```

## Create DNS record

The following records can be managed from the provider terraform:
- A
- AAAA
- CNAME
- TXT
- PTR
- SRV
- NS
- MX


Example for A record:

```
resource "dns_a_record_set" "www" {
  zone = "example.com."
  name = "www"
  addresses = [
    "192.168.0.1",
    "192.168.0.2",
    "192.168.0.3",
  ]
  ttl = 300
}
```

Run `terraform destroy` to delete it. 