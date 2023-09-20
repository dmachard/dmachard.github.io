---
title: "Play with extra DNStap field "
summary: "In this blog post, we will explore how to include additional metadata in your DNsStap logs, such as security flags, 
resolver IP addresses, and more."
date: 2023-09-20T00:00:00+01:00
draft: false
tags: ['dnstap', 'coredns', 'dnsdist']
pin: false
---

# Play with Extra DNStap field

In this blog post, we will explore how to include additional metadata in your [DNStap](https://dnstap.info/) logs, such as security flags, resolver IP addresses, and more.
The DNStap protocol offers an [extra](https://github.com/dnstap/dnstap.pb/blob/master/dnstap.proto#L37) field specifically designed for this purpose.

```c
// Extra data for this payload.
// This field can be used for adding an arbitrary byte-string annotation to
// the payload. No encoding or interpretation is applied or enforced.
optional bytes      extra = 3;
```

Let's begin by exploring how to implement this feature in CoreDNS.

## CoreDNS

Support for the extra field was introduced in CoreDNS starting from [v1.11.1](https://github.com/coredns/coredns/releases/tag/v1.11.1).
Below is an example of how to add the upstream IP (either 8.8.8.8 or 9.9.9.9) to the extra field.

First, create the coredns.conf file. This configuration sets up your CoreDNS instance to:

- Listen on port 53 for DNS queries.
- Enable metadata collection.
- Send dnstapto the remote collector 10.0.0.100 on port 6000 with full logging.
- Specify the identity as "coredns" for dnstap data.
- Include upstream addresses in extra field
- Cache DNS responses.
- Forward DNS queries to both 8.8.8.8 and 9.9.9.9 as upstream resolvers.

```bash
.:53 {
	metadata
        dnstap tcp://10.0.0.100:6000 full {
		identity coredns
		extra {/forward/upstream}
        }
	cache
        forward . 8.8.8.8 9.9.9.9
}
```

Next, start CoreDNS using the Docker image:

```bash
sudo docker run -d -p 8053:53/tcp -p 8053:53/udp --name=coredns -v $PWD/coredns.conf:/config.conf coredns/coredns:1.11.1 -conf /config.conf
```

To retrieve DNS logs with support for the extra field, you can utilize the [DNS-collector](https://github.com/dmachard/go-dnscollector)

Create the dnscollector.conf file. This configuration sets up DNS collector (dnscollector) to listen on port tcp 6000 for incoming dnstap data and send it to the console logger in a specified text format that includes the timestamp, identity, operation, query IP, extra field, query name (qname), and response code (rcode).

```yaml
multiplexer:
  collectors:
    - name: tap
      dnstap:
        listen-ip: 0.0.0.0
        listen-port: 6000

  loggers:
    - name: console
      stdout:
        mode: text
        text-format: "timestamp-rfc3339ns identity operation queryip extra qname rcode"

  routes:
    - from: [ tap ]
      to: [ console ]  
```

Now, start the DNS-collector using the Docker image:

```bash
sudo docker run -d -p 6000:6000/tcp --name=dnscollector -v $PWD/dnscollector.conf:/config.conf dmachard/go-dnscollector:v0.36.0 -config /config.conf
```

Execute the dig command to perform a DNS resolution using your CoreDNS instance:

```bash
dig @127.0.0.1 -p 8053 www.twitter.com
```

To view the DNScollector logs and observe the extra field, use the following command:

```bash
sudo docker logs dnscollector
```

You should see logs similar to the following, with the extra field equal to 8.8.8.8:53 and 9.9.9.9:53:

```bash
023-09-20T18:08:17.912209255Z coredns1 FORWARDER_QUERY 172.17.0.1 8.8.8.8:53 www.google.com NOERROR
2023-09-20T18:08:17.943256625Z coredns1 FORWARDER_RESPONSE 172.17.0.1 8.8.8.8:53 www.google.com NOERROR
2023-09-20T18:08:17.943348497Z coredns1 CLIENT_RESPONSE 172.17.0.1 8.8.8.8:53 www.google.com NOERROR

2023-09-20T18:08:51.271234557Z coredns1 CLIENT_QUERY 172.17.0.1 - www.twitter.com NOERROR
2023-09-20T18:08:51.271290121Z coredns1 FORWARDER_QUERY 172.17.0.1 9.9.9.9:53 www.twitter.com NOERROR
2023-09-20T18:08:51.294051653Z coredns1 FORWARDER_RESPONSE 172.17.0.1 9.9.9.9:53 www.twitter.com NOERROR
2023-09-20T18:08:51.294120571Z coredns1 CLIENT_RESPONSE 172.17.0.1 9.9.9.9:53 www.twitter.com NOERROR
```

## DNSdist

Next, continue into [DNSdist](https://dnsdist.org), a tool that offers greater flexibility in adding custom information to the extra field in dnstap messages.
Before getting started, within your working directory, generate certificates and a private key for use with the DNS over HTTPS (DoH) service:

```bash
openssl rand -base64 48 > passphrase.txt
openssl genrsa -aes128 -passout file:passphrase.txt -out server.key 2048
openssl req -new -passin file:passphrase.txt -key server.key -out server.csr -subj "/C=FR/O=krkr/OU=Domain Control Validated/CN=*.test.dev"
openssl rsa -in server.key -passin file:passphrase.txt -out doh.key
openssl x509 -req -days 36500 -in server.csr -signkey doh.key -out doh.crt
```

Now, create the `config_dnsdist.conf` file with the following Lua configuration.
This DNSdist configuration listen on DoH service with backends load balancing (google, cloudflare, quad9)
The lua functions (alterDnstapQuery, alterDnstapResponse, alterDnstapCachedResponse) are used to customize the extra field in dnstap messages.

```lua
addDOHLocal("0.0.0.0:443", "/etc/dnsdist/doh.crt", "/etc/dnsdist/doh.key", "/dns-query", {keepIncomingHeaders=true})
setACL({'0.0.0.0/0'})

newServer({address = "1.1.1.1", pool="poolA"})
newServer({address = "9.9.9.9", pool="poolB"})
newServer({address = "8.8.8.8", pool="poolB"})

function alterDnstapQuery(dq, tap)
  local ua = ""
  for key,value in pairs(dq:getHTTPHeaders()) do
    if key == 'user-agent' then
            ua = value
            break
    end
  end
  tap:setExtra(ua)
end

function alterDnstapResponse(dr, tap)
  tap:setExtra(dr.pool)
end

function alterDnstapCachedResponse(dr, tap)
  tap:setExtra("cached")
end

-- init dnstap remote collector
rl = newFrameStreamTcpLogger("192.168.1.17:6000")

-- rules for queries
addAction(AllRule(), DnstapLogAction("dnsdist1", rl, alterDnstapQuery))

addAction(ProbaRule(0.5), PoolAction("poolA"))
addAction(AllRule(), PoolAction("poolB"))

-- rules for replies
addResponseAction(AllRule(), DnstapLogResponseAction("dnsdist1", rl, alterDnstapResponse))
```

To start DNSdist using a Docker image, use the following command:

```bash
sudo docker run -d -p 8443:443/tcp -p 8083:8080 --name=dnsdist -v $PWD/:/etc/dnsdist/:ro powerdns/dnsdist-18:1.8.1
```

To test the DNSdist configuration, you can install and use the [q](https://github.com/natesales/q) client. 
Then, perform a DoH resolution using your DNSdist server with the following command:

```bash
./q www.google.fr A  @https://127.0.0.1:8443 --tls-no-verify
```

To view the DNScollector logs and observe the extra field, use the following command:

```bash
sudo docker logs dnscollector
```

You should see logs similar to the following, which include the "User-Agent" of the HTTP client or the "pool" used:

```bash
2023-09-20T20:20:39.423635597Z dnsdist1 CLIENT_RESPONSE 172.17.0.1 poolA www.google.fr NOERROR
2023-09-20T20:20:42.883887269Z dnsdist1 CLIENT_QUERY 172.17.0.1 Go-http-client/1.1 www.google.fr NOERROR
2023-09-20T20:20:42.899695506Z dnsdist1 CLIENT_RESPONSE 172.17.0.1 poolA www.google.fr NOERROR
2023-09-20T20:20:43.241895751Z dnsdist1 CLIENT_QUERY 172.17.0.1 Go-http-client/1.1 www.google.fr NOERROR
2023-09-20T20:20:43.254621667Z dnsdist1 CLIENT_RESPONSE 172.17.0.1 poolA www.google.fr NOERROR
2023-09-20T20:20:43.611780251Z dnsdist1 CLIENT_QUERY 172.17.0.1 Go-http-client/1.1 www.google.fr NOERROR
2023-09-20T20:20:43.628321105Z dnsdist1 CLIENT_RESPONSE 172.17.0.1 poolB www.google.fr NOERROR
```
