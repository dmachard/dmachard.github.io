---
title: "ELK installation on CentOS 7"
summary: "This post details how to install ELK (ElasticSearch, Logstash, Kibana and also Filebeat) on CentOS7."
date: 2020-06-21T00:00:00+01:00
draft: false
tags: ['elasticsearch', 'logs']
---

# ELK installation on CentOS 7

This post details how to install ELK (ElasticSearch, Logstash, Kibana and also Filebeat) on CentOS7.

## Installation ElasticSearch

```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

touch /etc/yum.repos.d/elasticsearch.repo
vim /etc/yum.repos.d/elasticsearch.repo


[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md

yum install --enablerepo=elasticsearch elasticsearch

systemctl daemon-reload
systemctl enable elasticsearch.service
```

## Configuration

```bash
vim /etc/elasticsearch/elasticsearch.yml
network.host: 0.0.0.0
discovery.type: single-node
```

## Start the service

```bash
systemctl start elasticsearch.service
systemctl status elasticsearch.service
```

## Sanity check

```bash
curl -X GET "localhost:9200"
```

## Logstash Installation with yum

```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

touch /etc/yum.repos.d/elasticsearch.repo
vim /etc/yum.repos.d/elasticsearch.repo


[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md

yum install java
yum install --enablerepo=elasticsearch logstash
systemctl enable logstash
```

## Configuration

```bash
vim /etc/logstash/logstash.yml
config.reload.automatic: true
```

## Start service

```bash
systemctl start logstash
systemctl status logstash
```

## Deploy new config

In the folder /etc/logstash/conf.d/ and test-it with the following command

```bash
/usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
```

# Kibana Installation

```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

touch /etc/yum.repos.d/kibana.repo
vim /etc/yum.repos.d/kibana.repo

[kibana-7.x]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md

yum install kibana
systemctl daemon-reload
systemctl enable kibana.service
```

## Starting Kibana

```bash
systemctl start kibana.service
systemctl status kibana.service
```

## Install apache in reverse proxy mode

```bash
yum install httpd

## Add a new virtual host

```bash
touch /etc/httpd.conf/kibana.conf
vim /etc/httpd.conf/kibana.conf

Listen 9090
<VirtualHost *:9090>
    ProxyPass / http://127.0.0.1:5601/
    ProxyPassReverse / http://127.0.0.1:5601/
</VirtualHost>
```
  
## Sanity check

From web browser, try to load kibana with http://x.x.x.x:9090

## Filebeat Installation

```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

touch /etc/yum.repos.d/elasticsearch.repo
vim /etc/yum.repos.d/elasticsearch.repo

[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md

yum install --enablerepo=elasticsearch filebeat
systemctl enable filebeat
```

## Configuration

```bash
vim /etc/filebeat/filebeat.yml

filebeat.inputs:
- type: log
enabled: true
paths:
    - /var/log/messages
fields:
    logtype: iptables

output.logstash:
    hosts: ["<ip_logstash>:5044"]
```