---
title: "Create Self-Signed Certificates with OpenSSL for testing purposes"
summary: "This post will describe how to generate Self-Signed Certificates"
date: 2023-09-27T00:00:00+01:00
draft: false
tags: ['security']
pin: false
---

# Create Self-Signed Certificates with OpenSSL

This post will describe how to generate Self-Signed Certificates. Remember that these certificates are not suitable for production use and should only be used in development and testing environments.

## Creating the Certificate Authority's certificate and key

Generate a private key for the CA (Certificate Authority):

```bash
openssl genrsa 2048 > ca.key
```

Create ca.conf file

```ini
[ req ]
prompt                 = no
distinguished_name     = req_distinguished_name

[ req_distinguished_name ]
countryName            = FR
stateOrProvinceName    = Normandie
localityName           = Caen
organizationName       = Home
organizationalUnitName = Lab
commonName             = ca.home.lab
emailAddress           = admin@home.lab
```

Generate the certificate for the CA:

```bash
openssl req -days 365 -new -x509 -nodes -key ca.key -out ca.crt --config ca.conf
```

## Creating the server's kertificate and key

Create server.conf file

```ini
[ req ]
prompt                 = no
distinguished_name     = req_distinguished_name
req_extensions         = req_ext

[ req_distinguished_name ]
countryName            = FR
stateOrProvinceName    = Normandie
localityName           = Caen
organizationName       = Home
organizationalUnitName = Lab
commonName             = server.home.lab
emailAddress           = admin@home.lab

[ req_ext ]
subjectAltName = DNS: server.home.lab, IP: 127.0.0.1
```

Generate the private key and CSR (Certificate Signing Request):

```bash
openssl req -newkey rsa:2048 -nodes -keyout server.key -out server.csr --config server.conf
```

Generate the certificate for the server:

```bash
openssl x509 -req -days 365 -in server.csr -out server.crt -CA ca.crt -CAkey ca.key -extensions req_ext -extfile server.conf
```

Show certificate

```bash
$ openssl x509 -text -noout -in server.crt
```

## Creating the client's certificate and key

Create client.conf file

```ini
[ req ]
prompt                 = no
distinguished_name     = req_distinguished_name

[ req_distinguished_name ]
countryName            = FR
stateOrProvinceName    = Normandie
localityName           = Caen
organizationName       = Home
organizationalUnitName = Lab
commonName             = client.home.lab
emailAddress           = admin@home.lab
```

Generate the private key and CSR:

```bash
openssl req -newkey rsa:2048 -nodes -keyout client.key -out client.csr --config client.conf
```

Generate the certificate for the client:

```bash
openssl x509 -req -days 365 -in client.csr -out client.crt -CA ca.crt -CAkey ca.key
```

## Verifying the certificates

Verify the server certificate:

```bash
openssl verify -CAfile ca.crt server.crt
server.crt: OK
```

Verify the client certificate:

```bash
openssl verify -CAfile ca.crt client.crt
client.crt: OK
```
