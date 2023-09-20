---
title: "Secure DNSTAP streams with TLS encryption in CoreDNS"
summary: "In this post, you will see how to enable TLS encryption for outgoing dnstap streams with CoreDNS"
date: 2023-08-28T00:00:00+01:00
draft: false
tags: ['coredns', 'dnstap', 'dns', 'logs', 'security']
pin: true
---

# Secure DNSTAP streams with TLS encryption in CoreDNS

In this post, you will see how to enable TLS encryption for outgoing [dnstap](https://dnstap.info/) streams with [CoreDNS](https://coredns.io/). Before proceeding, please refer to [How to enable it on main dns servers](https://dmachard.github.io/posts/0001-dnstap-testing/#coredns) to enable DNStap in basic way.

> Note: TLS support in CoreDNS is available starting from version [1.11](https://github.com/coredns/coredns/releases/tag/v1.11.0)

## Enable Secure DNSTAP with CoreDNS

Begin by creating a dedicated working folder for your secure DNStap configurations:

```bash
mkdir securednstap
cd securednstap
```

Create a file named `config_coredns.conf` in your working folder with the following contents:

```ini
.:53 {
        dnstap tls://192.168.1.200:6000 full {
                skipverify
        }
        forward . 8.8.8.8:53
}
```

Here's what the configuration does:

- `tls://` is used to specify TLS encryption.
- `192.168.1.200:6000` should be replaced with the IP/port address of your [DNStap collector](https://github.com/dmachard/go-dnscollector).
- `skipverify` is used to skip TLS verification. **Do not use this setting in a production environment**

Execute CoreDNS with the following command:

```bash
sudo docker run -d -p 1053:53/tcp -p 1053:53/udp --name=coredns -v $PWD/config_coredns.conf:/config.conf coredns/coredns:1.11.1 -conf /config.conf
```

Check the CoreDNS logs to verify the configuration:

```bash
sudo docker logs coredns
[ERROR] plugin/dnstap: No connection to dnstap endpoint: dial tcp 192.168.1.200:6000: connect: connection refused
.:53
CoreDNS-1.11.1
linux/amd64, go1.20.7, ae2bbc2
```

> At this stage, you might encounter an error indicating that there is no connection to the DNStap collector. This is expected because the remote DNStap collector is not yet running.

## Running DNS-collector

To collect DNS logs with a secure DNStap stream, you can use the [DNS-collector](https://github.com/dmachard/go-dnscollector).
Here's how to set it up:

Within your working directory, generate certificates and a private key for the server side:

```bash
openssl rand -base64 48 > passphrase.txt
openssl genrsa -aes128 -passout file:passphrase.txt -out server.key 2048
openssl req -new -passin file:passphrase.txt -key server.key -out server.csr -subj "/C=FR/O=krkr/OU=Domain Control Validated/CN=*.test.dev"
openssl rsa -in server.key -passin file:passphrase.txt -out dnscollector.key
openssl x509 -req -days 36500 -in server.csr -signkey dnscollector.key -out dnscollector.crt
```

Create a file named `config_dnscollector.conf` in your working folder with the following content.
This configuration sets up the DNS-collector with TLS support and specifies the certificate and key files.

```yaml
global:
  trace:
    verbose: true

multiplexer:
  collectors:
    - name: securednstap
      dnstap:
        listen-ip: 0.0.0.0
        listen-port: 6000
        tls-support: true
        cert-file: "/custom/dnscollector.crt"
        key-file: "/custom/dnscollector.key"

  loggers:
    - name: console
      stdout:
        mode: text

  routes:
    - from: [securednstap]
      to: [console]
```

Run the DNS-collector using the following command:

```bash
sudo docker run -d -p 6000:6000/tcp --name=dnscollector -v $PWD/:/custom dmachard/go-dnscollector:v0.34.0 -config /custom/config_dnscollector.conf
```

Check the DNS-collector logs to ensure everything is running properly.
You should see log entries indicating that the DNS-collector is running and listening for secured DNStap connections.

```bash
INFO: 2023/08/28 07:57:39.848987 main - version v0.34.0
INFO: 2023/08/28 07:57:39.849738 main - starting dns-collector...
INFO: 2023/08/28 07:57:39.849742 main - loading loggers...
INFO: 2023/08/28 07:57:39.850252 [console] logger=stdout - enabled
INFO: 2023/08/28 07:57:39.887316 main - loading collectors...
INFO: 2023/08/28 07:57:39.887497 [securednstap] collector=dnstap - enabled
INFO: 2023/08/28 07:57:39.887506 main - routing: collector[securednstap] send to logger[console]
INFO: 2023/08/28 07:57:39.887701 main - running...
INFO: 2023/08/28 07:57:39.887713 [securednstap] collector=dnstap - starting collector...
INFO: 2023/08/28 07:57:39.887810 [securednstap] collector=dnstap - running in background...
INFO: 2023/08/28 07:57:39.887815 [securednstap] collector=dnstap - tls support enabled
INFO: 2023/08/28 07:57:39.887849 [console] logger=stdout - running in background...
INFO: 2023/08/28 07:57:39.888050 [console] logger=stdout - ready to process
INFO: 2023/08/28 07:57:39.888760 [securednstap] collector=dnstap - is listening on tls://[::]:6000
INFO: 2023/08/28 07:57:40.774579 [securednstap] collector=dnstap#1 - new connection from 172.17.0.1:49980
INFO: 2023/08/28 07:57:40.774591 [securednstap] processor=dnstap#1 - initialization...
INFO: 2023/08/28 07:57:40.774909 [securednstap] processor=dnstap#1 - waiting dns message to process...
INFO: 2023/08/28 07:57:40.778959 [securednstap] collector=dnstap#1 - receiver framestream initialized
```

## Testing Secure DNStap

To test your secure DNStap stream, perform a DNS resolution using CoreDNS:

```bash
dig -p 1053 www.google.com +short +tcp
172.217.18.196
```

Next, check if your DNStap messages are being received by the DNS-collector:
You should see log entries indicating that the DNS-collector has received and processed the DNS queries and responses from the secure DNStap stream.

```bash
sudo docker logs dnscollector
2023-08-28T07:59:24.028882855Z 9116ac214d22 CLIENT_QUERY NOERROR 172.17.0.1 51492 IPv4 TCP 55b www.google.com A 0.000000
2023-08-28T07:59:24.028911293Z 9116ac214d22 FORWARDER_QUERY NOERROR 172.17.0.1 51492 IPv4 TCP 55b www.google.com A 0.000000
2023-08-28T07:59:24.052002916Z 9116ac214d22 FORWARDER_RESPONSE NOERROR 172.17.0.1 51492 IPv4 TCP 73b www.google.com A 0.000000
2023-08-28T07:59:24.052069329Z 9116ac214d22 CLIENT_RESPONSE NOERROR 172.17.0.1 51492 IPv4 TCP 73b www.google.com A 0.000000
```