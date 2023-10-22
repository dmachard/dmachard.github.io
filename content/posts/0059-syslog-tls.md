---
title: "Enable TLS encryption and mutual authentication with syslog-ng"
summary: "Enhance the security of your log management system by enabling TLS encryption and mutual authentication"
date: 2023-10-22T00:00:00+01:00
draft: false
tags: ['security', 'syslog-ng']
pin: false
---

# Enable TLS encryption and mutual authentication with syslog-ng

By following this guide, you can enhance the security of your log management system by enabling TLS encryption and mutual authentication with `syslog-ng`. This ensures that your log data remains confidential and trustworthy, even in a potentially insecure environment.

## Prepare working directory

Before we begin, make sure you have the following prerequisites in place:

- Docker: You'll need Docker installed on your system to run syslog-ng in a container.
- A working directory: Create a directory for your syslog-ng configuration and logs.

> This guide test is based on the docker [https://hub.docker.com/r/linuxserver/syslog-ng](linuxserver/syslog-ng) image version 4.1

You can do this with the following commands:

```bash
mkdir syslogng_tls
mkdir syslogng_tls/config
mkdir syslogng_tls/log
```

## Configuration

1; Create self-signed certificates

In this guide, you'll need to create self-signed certificates and place them in the conf folder.
For a detailed guide on how to create self-signed certificates using OpenSSL, you can refer to this tutorial: [https://dmachard.github.io/posts/0057-create-self-certificate/](Create Self-Signed Certificates with OpenSSL)

To avoid the error following error in syslog-ng: `Invalid certificate found in chain, basicConstraints.ca is unset in non-leaf certificate`

Generate your Certificate Authority with 

```ini
[ req ]
x509_extensions               = v3_ca

[ v3_ca ]
basicConstraints         = CA:TRUE
```

2; Create syslog-ng Configuration

Now, let's create the syslog-ng.conf configuration file. Below is a sample configuration that sets up syslog-ng to use TLS encryption and listens on port 6514:

```ini
@version: 4.1

source s_network_tls {
  syslog(
        transport(tls)
        port(6514)
        tls (
           cert-file("/config/server.crt")
           key-file("/config/server.key")
           peer-verify(optional-untrusted)
        )
  );
};

destination d_local {
  file("/var/log/messages-kv.log" template("$ISODATE $HOST $(format-welf --scope all-nv-pairs)\n") frac-digits(3));
};

log {
  source(s_network_tls);
  destination(d_local);
};
```

3; Create Docker Compose

Next, create a Docker Compose file, `docker-compose.yml`, to deploy the syslog-ng container:

> Docker file is on github https://github.com/linuxserver/docker-syslog-ng
> All config are available in the following github [github.com/dmachard/docker-stack-syslog-ng](repository)

```bash
version: "3.8"

services:
  syslog-ng:
    image: lscr.io/linuxserver/syslog-ng:4.1.1
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - ./config:/config
      - ./log:/var/log
    ports:
      - 6514:6514/tcp
    restart: unless-stopped
```

## Deploy syslog-ng with Docker Compose

Deploy syslog-ng with Docker Compose

```bash
$ sudo docker compose up -d
[+] Running 1/1
 ✔ Container syslogng-compose-syslog-ng-1  Started
```

You can check the status of your Docker containers with:

```bash
$ sudo docker compose ls
NAME                STATUS              CONFIG FILES
syslogng-compose    running(1)          syslogng-compose/docker-compose.yml
```

## Sending encrypted logs

To send encrypted logs to your syslog server, you can use the socat command. Here's an example:

```bash
$ printf "107 <30>1 2023-10-17T21:25:15+02:00 hostname /usr/bin/binary 78175 tag - This is a sample log message over TLS." | socat - openssl:127.0.0.1:6514,verify=0
```

You can monitor the incoming logs in the messages-kv.log file.

```bash
$ tail -f messages-kv.log
2023-10-17T21:25:15.000+02:00 denis-laptop HOST=denis-laptop HOST_FROM=denis-laptop MESSAGE="This is a sample log message over TLS." MSGID=tag PID=78175 PROGRAM=/usr/bin/binary SOURCE=s_network_tls
```

## Mutual TLS authentication

For enhanced security, you can enable mutual TLS authentication (mTLS) and restrict TLS to version 1.3.
Update your syslog-ng configuration as follows:

```ini
source s_network_tls {
  syslog(
        transport(tls)
        port(6514)
        tls (
           cert-file("/config/server.crt")
           key-file("/config/server.key")
           ca-file("/config/ca.crt")
           peer-verify(required-trusted)
           ssl-options(no-tlsv12)
        )
  );
};
```

After making this change, restart your syslog-ng container.

Use nmap command to check the TLS version

```bash
$ nmap --script ssl-enum-ciphers -p 6514 127.0.0.1
Starting Nmap 7.93 ( https://nmap.org ) at 2023-10-22 16:17 CEST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00011s latency).

PORT     STATE SERVICE
6514/tcp open  syslog-tls
| ssl-enum-ciphers: 
|   TLSv1.3: 
|     ciphers: 
|       TLS_AKE_WITH_AES_256_GCM_SHA384 (ecdh_x25519) - A
|       TLS_AKE_WITH_CHACHA20_POLY1305_SHA256 (ecdh_x25519) - A
|       TLS_AKE_WITH_AES_128_GCM_SHA256 (ecdh_x25519) - A
|     cipher preference: server
|_  least strength: A

Nmap done: 1 IP address (1 host up) scanned in 0.25 seconds
```

To test mutual TLS authentication, try sending a log message without providing the client certificate and key. 

```bash
$ printf "107 <30>1 2023-10-17T21:25:15+02:00 hostname /usr/bin/binary 78175 tag - This is a sample log message over TLS." | socat - openssl:127.0.0.1:6514,verify=0
```

You should receive an error message indicating that the peer did not return a certificate.

```bash
$ tail -f config/log/current
2023-10-22 17:08:42.165093663  [2023-10-22T17:08:42.165054] Syslog connection accepted; fd='13', client='AF_INET(172.21.0.1:46488)', local='AF_INET(0.0.0.0:6514)'
2023-10-22 17:08:42.170333871  [2023-10-22T17:08:42.170290] SSL error while reading stream; tls_error='error:0A0000C7:SSL routines::peer did not return a certificate', location='/config/syslog-ng.conf:7:4'
2023-10-22 17:08:42.170373178  [2023-10-22T17:08:42.170322] Error reading RFC6587 style framed data; fd='13', error='Connection reset by peer (104)'
2023-10-22 17:08:42.170387705  [2023-10-22T17:08:42.170369] Syslog connection closed; fd='13', client='AF_INET(172.21.0.1:46488)', local='AF_INET(0.0.0.0:6514)'
```

Retry to send a log message with mutual authentication, provide the client certificate and key, like this:

```bash
printf "107 <30>1 2023-10-17T21:25:15+02:00 hostname /usr/bin/binary 78175 tag - This is a sample log message over TLS." | socat - openssl:127.0.0.1:6514,cert=config/client.crt,key=config/client.key,verify=0
```

Your log should now be successfully received and logged by syslog-ng.

```bash
$ tail -f messages-kv.log
2023-10-17T21:25:15.000+02:00 denis-laptop .tls.x509_cn=client.home.lab .tls.x509_o=Home .tls.x509_ou=Lab HOST=denis-laptop HOST_FROM=denis-laptop MESSAGE="This is a sample log message over TLS." MSGID=tag PID=78175 PROGRAM=/usr/bin/binary SOURCE=s_network_tls
```
