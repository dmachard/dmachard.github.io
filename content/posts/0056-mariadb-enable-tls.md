---
title: "Setting up SSL/TLS encryption with official MariaDB docker image"
summary: "This post will describe the steps to configure SSL/TLS encryption for a MariaDB container running in Docker."
date: 2023-09-27T05:00:00+01:00
draft: false
tags: ['mariadb', 'security']
pin: false
---

# Setting up SSL/TLS encryption with official MariaDB docker image

This post will describe the steps to configure SSL/TLS encryption for a MariaDB container running in Docker.
with the official image available in [dockerhub](https://hub.docker.com/_/mariadb).

## Prequisites

Before enabling TLS encryption, you need to generate server certificates and a private key.
Follow the steps outlined in the [Create Self-Signed Certificates and Keys with OpenSSL](https://dmachard.github.io/posts/0057-create-self-certificate/) guide to accomplish this.

## Create a Custom SSL Configuration

Create a custom MariaDB SSL configuration file named `1-server-ssl.cnf`, and add the following content to it:

```ini
[mariadbd]
ssl-ca = /etc/mysql/conf.d/ca-cert.pem
ssl-cert = /etc/mysql/conf.d/server-cert.pem
ssl-key = /etc/mysql/conf.d/server-key.pem
```

This configuration file specifies the paths to the SSL certificate authority (CA), server certificate, and server key to use.

## Run the MariaDB container

Now, run the MariaDB container the custom SSL config mounted to `/etc/mysql/conf.d`:

```bash
docker run --name mariadb -v /my/custom:/etc/mysql/conf.d -e MARIADB_ROOT_PASSWORD=pw-d mariadb:latest
```

## Test the SSL/TLS Connection

To verify that SSL/TLS encryption is working, you can connect to the MariaDB server using SSL. Use the following command:

```bash
mariadb --ssl -h 127.0.0.1 -uroot -plabpwd
```

And run the following SQL queries to confirm that SSL/TLS is properly configured:

```bash
MariaDB [(none)]> SHOW VARIABLES LIKE 'have_ssl';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| have_ssl      | YES   |
+---------------+-------+
1 row in set (0.003 sec)

MariaDB [(none)]> SHOW SESSION STATUS LIKE "ssl_cipher";
+---------------+------------------------+
| Variable_name | Value                  |
+---------------+------------------------+
| Ssl_cipher    | TLS_AES_256_GCM_SHA384 |
+---------------+------------------------+
1 row in set (0.003 sec)
```

This output confirm that SSL/TLS encryption is enabled with the encryption cipher in use for the connected session.
