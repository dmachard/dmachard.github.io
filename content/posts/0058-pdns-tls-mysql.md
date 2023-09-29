---
title: "How to enable TLS encryption with gmysql PowerDNS backend "
summary: "In this post, we will see how to enable TLS encryption with authentication between PowerDNS authoritative server and remote MySQL database."
date: 2023-09-29T00:00:00+01:00
draft: false
tags: ['security', 'powerdns', 'mysql']
pin: false
---

# How to enable TLS encryption with gmysql PowerDNS backend

In this post, we will see how to enable TLS encryption with authentication between PowerDNS authoritative server and remote MySQL database.

## Enable TLS encryption - One-way mode

> One-way TLS means that the client verifies that the certificate belongs to the server.

First all, enable encryption on your MariaDB instance with `TLSv1.3` only

```ini
[mariadbd]
ssl-ca = /etc/mysql/conf.d/ca.crt
ssl-cert = /etc/mysql/conf.d/server.crt
ssl-key = /etc/mysql/conf.d/server.key
tls_version = TLSv1.3
```

And grant the powerdns user with `require ssl` 

```sql
ALTER USER 'pdns_user'@'%' REQUIRE SSL;
```

Now update the `pdns.conf` config by added the `gmysql-group` key

```ini
launch=gmysql
gmysql-host=172.16.0.10
gmysql-dbname=pdns
gmysql-user=pdns_user
gmysql-password=pdns_secret
gmysql-group=pdns
```

Finnaly add the `/etc/my.cnf` file and provides the certificate and private key of the client.

```ini
ssl-cert = /etc/powerdns/client.crt
ssl-key = /etc/powerdns/client.key
ssl_ca = /etc/powerdns/ca.crt
ssl-verify-server-cert
# force TLS version for client too
#tls_version = TLSv1.2,TLSv1.3
```

Restart your pdns instance and check if the connection is successful and properly enforced with TLS.

```bash
$ sudo docker exec docker-stack-dns-ns-1 pdnsutil backend-cmd gmysql 'show session status like "ssl_cipher"'
== show session status like "ssl_cipher"
'Ssl_cipher'	'TLS_AES_256_GCM_SHA384'
```

## Enable Two-Way TLS

> Two-way TLS means that both the client can be authenticated by the server. to do that the client must provide a X509 certificate to the server.

Grant the powerdns user with  `require subject`

```sql
ALTER USER 'pdns_user'@'%' REQUIRE SUBJECT '/C=AU/ST=Some-State/O=Internet Widgits Pty Ltd/CN=client.hello.world';
```

Restart your pdns, authentication should be OK.

If you have the `access denied` error like below take a look to your db logs

```bash
docker-stack-dns-ns-1  | Sep 28 19:10:34 gmysql Connection failed: Unable to connect to database: ERROR 1045 (28000): Access denied for user 'pdns_user'@'172.16.0.30' (using password: YES)
```

The subject mismatch should appears

```bash
docker-stack-dns-db-1  | 2023-09-28 19:10:34 6 [Note] X509 subject mismatch: should be '/CN=client.hello.world' but is '/C=AU/ST=Some-State/O=Internet Widgits Pty Ltd/CN=client.hello.world'
docker-stack-dns-db-1  | 2023-09-28 19:10:34 6 [Warning] Access denied for user 'pdns_user'@'172.16.0.30' (using password: YES)
```

> Some additionals links
>
> - https://dmachard.github.io/posts/0056-mariadb-enable-tls/
> - https://mariadb.com/kb/en/securing-connections-for-client-and-server/
> - https://mariadb.com/docs/server/security/data-in-transit-encryption/enterprise-server/enable-tls/
> - https://mariadb.com/kb/en/securing-connections-for-client-and-server/#requiring-tls
