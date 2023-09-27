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

Generate the certificate for the CA:

```bash
openssl req -new -x509 -nodes -days 365 -key ca.key -out ca.crt
```

## Creating the server's kertificate and key

Generate the private key and CSR (Certificate Signing Request):

```bash
openssl req -newkey rsa:2048 -nodes -days 365 -keyout server.key -out server.csr
```

Generate the certificate for the server:

```bash
openssl x509 -req -days 365 -rand_serial -in server.csr -out server.crt -CA ca.crt -CAkey ca.key
```

## Creating the client's certificate and key

Generate the private key and CSR:

```bash
openssl req -newkey rsa:2048 -nodes -days 365 -keyout client.key -out client.csr
```

Generate the certificate for the client:

```bash
openssl x509 -req -days 365 -rand_serial -in client.csr -out client.crt -CA ca.crt -CAkey ca.key
```

## Verifying the certificates

Verify the server certificate:

```bash
openssl verify -CAfile ca.crt server.crt
```

Verify the client certificate:

```bash
openssl verify -CAfile ca.crt client.crt
```
