---
title: "Authoritary PowerDNS installation on CentOS"
summary: "PowerDNS authoritary servers installation procedure"
date: 2020-06-21T00:00:00+01:00
draft: false
tags: ['dns', 'powerdns', 'authoritary']
---

# Authoritary PowerDNS installation on CentOS

PowerDNS authoritary servers installation on **CentOS 7* with **sqlite** database

## Installation

```bash
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum -y install epel-release
yum --enablerepo=epel install luajit libsodium
yum --enablerepo=epel install jq

curl -o /etc/yum.repos.d/powerdns-auth-42.repo https://repo.powerdns.com/repo-files/centos-auth-42.repo
yum install pdns pdns-tools pdns-backend-sqlite


wget https://raw.githubusercontent.com/PowerDNS/pdns/master/modules/gsqlite3backend/schema.sqlite3.sql


mkdir /var/db/pdns
sqlite3 /var/db/pdns/pdns.db
.read schema.sqlite3.sql
.quit

chown -R pdns:pdns /var/db/pdns
```

## Configuration

```bash
vim /etc/pdns/pdns.conf

version-string=anonymous
setuid=pdns
setgid=pdns

local-ipv6=

launch=gsqlite3
gsqlite3-database=/var/db/pdns/pdns.db

webserver=yes
webserver-address=0.0.0.0
webserver-port=8081
webserver-allow-from=127.0.0.0/8,10.0.0.0/24

api=on
api-key=test

master=yes

dnsupdate=yes

local-address=0.0.0.0
local-port=5300

log-dns-details=on
log-dns-queries=yes
log-timestamp=yes
loglevel=4

security-poll-suffix=
allow-dnsupdate-from=
```

## Start server

```bash
systemctl enable pdns.service
systemctl start pdns.service
```

## Manage configuration

Manage configuration with pdnsutil command

- Add a new zone

```bash
pdnsutil create-zone home.local
pdnsutil create-zone home.local ns1.home.local

pdnsutil add-record home.local ns1 A 3600 10.0.0.71
```

- Add a new resource record

```bash
pdnsutil add-record home.local ea A 300 10.0.0.72
```

- List all zones

```bash
pdnsutil list-all-zones
```

- Delete a zone

```bash
pdnsutil delete-zone home.local
```

- Delete a resource record
    
```bash
pdnsutil delete-rrset home.local ea A
```

- Update resource record

```bash
pdnsutil replace-rrset home.local ea A 3600 10.0.0.1
```

- Edit a zone

```bash
export EDITOR=vim
pdnsutil edit-zone home.local
```

## Cache purge entries 

```bash
pdns_control purge test1.home.local
```

## DNS-UPDATE (RFC2136)

###  Activation 

- Generate the tsig key for the domain to securize

```bash
pdnsutil generate-tsig-key homelocal_update hmac-md5
```

- Activate the key in the domain

```bash
pdnsutil set-meta home.local TSIG-ALLOW-DNSUPDATE homelocal_update
```

- Authorize just some network to make update in the zone

```bash
pdnsutil set-meta home.local ALLOW-DNSUPDATE-FROM 10.0.0.0/24
```

- Activate notification to slave server on each update

```bash
pdnsutil set-meta home.local NOTIFY-DNSUPDATE 1
```

- Display all TSIG keys

```bash
pdnsutil list-tsig-keys
```

### Testing DNS UPDATE

- Create the file nsupdate.txt with the following content

```bash
server <pdns_ip_address> <pdns_port>
zone home.local
update add test1.home.local 3600 A 10.0.0.140
key <key_name> <tsig_key>
show
send
```

- Execute the nsupdate command to make the update in the zone

```bash
nsupdate -v dnsupdate.txt
```

## GSLB feature

- Update the configuration to activate LUA 

```bash
enable-lua-records=yes
```

- Return time with TXT type

```bash
pdnsutil add-record time.gslb.test time LUA 0 'TXT "os.date()"'

dig @10.0.0.235 -t txt time.gslb.test +short
"Mon Oct 14 16:05:06 2019"
```

- Return random IP with round robin 

```bash
pdnsutil add-record gslb.test roundrobin LUA 0 "A 
\"pickrandom({'10.0.0.1','10.0.0.2','10.0.0.3','10.0.0.4'})\""
```

- Return specific IP accord to the result of TCP monitor. If all backends are down, then the complete list is returned otherwhise just one server is returned in random mode.

```bash
pdnsutil add-record gslb.test tcp LUA 0 "A 
\"ifportup(80,{'10.0.0.1', '10.0.0.2'}, {selector='random',backupSelector='all'})\""
```

- Return specific IP accord to the result of HTTP monitor. An URL is currently considered UP if the HTTP response codeis equal to 200. The checks are performed sequentially, with aminimum delay of 5 seconds.

```bash
pdnsutil add-record gslb.test http LUA 0 "A 
\"ifurlup('http://home.local', {'10.0.0.1','10.0.0.2'}, 
{selector='random', backupSelector='all',stringmatch='UP'})\""
```

## Play with REST API

- Get all zones 

```bash
curl -s -H 'X-API-Key: dns' http://127.0.0.1:8082/api/v1/servers/localhost
```

- Get a specific zone

```bash
curl -s -H 'X-API-Key: dns' http://127.0.0.1:8082/api/v1/servers/localhost/zones/home.local
```

- Generate a TSIG key

```bash
curl -s -X POST -H "X-API-Key: dns" -H "Content-Type: application/json" 
-d '{"name": "mytsigkey2", "algorithm": "hmac-sha256"}' 
http://127.0.0.1:8082/api/v1/servers/localhost/tsigkeys
```

- Get metadata for a specific zone

```bash
curl -s -H 'X-API-Key: dns' http://127.0.0.1:8082/api/v1/servers/localhost/zones/home.local/metadata
```

- Delete metadata

```bash
curl -s -X DELETE -H 'X-API-Key: dns'
http://127.0.0.1:8082/api/v1/servers/localhost/zones/home.local/metadata/ALLOW-DNSUPDATE-FROM
```
     
- Add metadata

```bash
curl -s -X POST -H 'X-API-Key: dns' -H "Content-Type: application/json" -d "@restapi_metadata.json"
http://127.0.0.1:8082/api/v1/servers/localhost/zones/home.local/metadata
```

- Update RR 

```bash
curl -s -X PATCH -H "X-API-Key: dns" -d '{"rrsets": [{"name": "test1.home.local.", "type": "A",
"changetype": "REPLACE", "records": [{"content": "3.100.10.72", "disabled": false}],
"ttl": 3600}]}' http://10.0.0.235:8082/api/v1/servers/localhost/zones/home.local.
```