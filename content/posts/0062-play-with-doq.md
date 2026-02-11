---
title: "Play with DNS over QUIC and DoH3"
summary: "Some notes to test DoQ"
date: 2024-10-09T00:00:00+01:00
draft: false
tags: ['dns', 'doq', 'quic', 'dnsdist', 'powerdns']
pin: false
---

# Play with DNS over QUIC and DoH3

## Enable DoQ/DoH3 on dnsdist

This guide shows how to configure dnsdist to support DNS over QUIC (DoQ) and DNS over HTTP/3 (DoH3). 

### Step 1: Create Certificates

You'll need a valid SSL certificate and private key to enable DoQ and DoH3. You can either create self-signed certificates or use ones from a trusted CA.

For example:

    cert.pem: SSL certificate
    key.pem: SSL private key

### Step 2: Create the dnsdist Configuration

In this configuration, we bind dnsdist to support standard DNS on port 53, DoQ on port 853, and DoH3 on port 443.

Here’s a sample configuration:

```bash
setLocal("0.0.0.0:53", {})

addDOQLocal('0.0.0.0:853', '/etc/dnsdist/cert.pem', '/etc/dnsdist/key.pem')
addDOH3Local('0.0.0.0:443', '/etc/dnsdist/cert.pem', '/etc/dnsdist/key.pem')

newServer({address = "8.8.8.8", pool = "default" })

addAction(AllRule(),PoolAction("default"))
```

This configuration does the following:

- Listens for regular DNS on 0.0.0.0:53.
- Enables DoQ on port 853 and DoH3 on port 443, both using the specified certificates.
- Uses 8.8.8.8 as the upstream DNS server.
- Routes all DNS traffic to the default pool.

### Step 3: Create the Docker Compose File

Here’s a basic docker-compose.yml file to run dnsdist with DoQ and DoH3 enabled:

```bash
services:
  dnsdist:
    image: powerdns/dnsdist-19:1.9.4
    volumes:
      - ./cert.pem:/etc/dnsdist/cert.pem
      - ./key.pem:/etc/dnsdist/key.pem
      - ./dnsdist.conf:/etc/dnsdist/conf.d/dnsdist.conf
    ports:
      - "853:853/udp"
      - "443:443/udp"
      - "5553:53/udp"
      - "5552:52/tcp"
      - "443:443/tcp"
      - "8083:8080/tcp"
```

This file does the following:
- Maps your local cert.pem, key.pem, and dnsdist.conf files to the correct locations inside the container.
- Exposes DoQ (port 853) and DoH3 (port 443) for both UDP and TCP.
- Exposes regular DNS on port 5553.

### Step 4: Start dnsdist

```bash
sudo docker compose up -d
```

### Step 5: View Logs

```bash
$ sudo docker compose logs -f
dnsdist-1  | dnsdist 1.9.4 comes with ABSOLUTELY NO WARRANTY.
dnsdist-1  | Added downstream server 8.8.8.8:53
dnsdist-1  | Listening on 0.0.0.0:53
dnsdist-1  | Listening on 0.0.0.0:853 for DoQ
dnsdist-1  | Listening on 0.0.0.0:443 for DoH3
```

## Install DoQ/DoH3 Client

To test the DoQ and DoH3 services, you’ll need a client that supports these protocols. 
You can use the q client, which is a simple utility for testing DoQ and DoH.

Download and install the client from the [q](https://github.com/natesales/q) GitHub repository.


## DoQ Testing
 
You can now test the DoQ setup using the following command:

```bash
$ ./q www.google.com A @quic://127.0.0.1:853 --tls-insecure-skip-verify
```

Sample output:

```bash
www.google.com. 3m2s A 216.58.214.164
```

## DoH3 Testing

To test DoH3, use the following command:

```bash
./q www.google.com A @https://127.0.0.1:443 --http3 --tls-insecure-skip-verify
```

Sample output:

```bash
www.google.com. 3m7s A 172.217.20.164
```