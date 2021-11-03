---
title: "Ntpd: How to get statistics usage with ELK"
date: 2020-06-20T00:00:00+01:00
draft: false
tags: ["ntp", 'logs']
---

In this post, I will look at ntpd server  to extract some ntp statitics.

## Table of contents
* [Activate ntpd logs](#activate-ntpd-logs)
	* [peerstats documentation](#peerstats-documentation)
 	* [sysstats documentation](#sysstats-documentation)
* [Counting NTP clients](#counting-ntp-clients)
* [Expected logs](#expected-logs)
* [Export Logs to ELK](#export-logs-to-elk)
	* [Deploy Filebeat](#deploy-filebeat)
	* [Configure Logstash](#configure-logstash)
* [Examples of dashboard on Kibana](#examples-of-dashboard-on-kibana)

## Activate ntpd logs

Update the ntpd configuration

      vim /etc/ntp.conf
      statsdir /var/log/ntpstats/
      statistics peerstats sysstats
      filegen peerstats file peerstats.log type day enable
      filegen sysstats file sysstats.log type none enable
 
Restart the daemon

      systemctl restart ntpd
      
### peerstats documentation

Record peer statistics. 
An event occurs upon an update from either another NTP server 

| Units | Description | 
|-------|-------| 
| MJD | date | 
| s | time past midnight | 
| IP | source address | 
| hex | status word | 
| s | clock offset | 
| s | roundtrip delay | 
| s | dispersion | 
| s | jitter | 

    tailf /var/log/ntpstats/peerstats
    58927 1293.735 91.224.149.41 941a -0.031646296 0.047783430 0.019218732 0.002787419
    58927 1315.710 129.250.35.250 945a -0.033762728 0.027946319 0.019098537 0.001580388
    58927 1848.736 193.141.27.6 941a -0.034690903 0.054624853 0.015236537 0.001218647
    58927 2006.714 92.222.117.115 961a -0.033217220 0.031581741 0.015059831 0.001349028

| | Definition |
|----|--------|
| delay | Shows the round trip time (RTT) of your computer communicating with the remote server. |
| offset |  Using root mean squares, and shows how far off your clock is from the reported time the server gave you. It can be positive or negative. |
| jitter | Showing the root mean squared deviation of your offsets. |

### sysstats documentation

Each hour one line is appended to the sysstats file set in the following format

| Units | Description |
|-------|-------------|
| MJD | date |
| s  | time past midnight |
| s  | time since reset |
|  | packets received |
| | packets generated |
| | current versions |
| | old version |
| | access denied |
| | bad length or format |
| | bad authentication |
| | declined |
| | rate exceeded |
| | kiss-o'-death packets sent |
 
     tailf /var/log/ntpstats/sysstats
     58927 52035.251 3599 225 205 225 0 0 0 0 0 0 0
     58927 55635.257 3600 88 88 88 0 0 0 0 0 0 0
     58927 59235.313 3600 49 49 49 0 0 0 0 0 0 0
  
## Counting NTP clients

The ntpd does not give you how many clients it serves.

Install iptables 

	yum remove firewalld && yum install iptables-services
	systemctl enable iptables.service

Add the rule to log UDP port 123 traffic

	/sbin/iptables -A INPUT -p udp --dport 123 -j LOG --log-prefix='[NTP] ' --log-level debug
	/usr/libexec/iptables/iptables.init save

Update rsyslog to log NTP traffic in specific file

	touch /etc/rsyslog.d/00-iptables.conf
	vim /etc/rsyslog.d/00-iptables.conf
	:msg,contains,"[NTP] " /var/log/ntpstats/iptables.log

Restart rsyslog

	service rsyslog restart
	
Example of log

	# tailf /var/log/ntpstats/iptables.log
	Mar 19 06:56:10 localhost kernel: [NTP] IN=enp0s3 OUT= MAC=08:00:27:96:57:49:a0:40:a0:8d:29:95:08:00 \
	SRC=185.242.56.3 DST=10.0.0.33 LEN=76 TOS=0x00 PREC=0x00 TTL=44 ID=56768 \
	DF PROTO=UDP SPT=123 DPT=123 LEN=56
	
## Expected logs

      [root@localhost ntpstats]# ll
      total 740
      -rw-------. 1 root root 669741 Mar 22 08:42 iptables.log
      -rw-r--r--. 2 ntp  ntp    3549 Mar 22 07:14 peerstats.log
      -rw-r--r--. 1 ntp  ntp   32829 Mar 21 00:58 peerstats.log.20200320
      -rw-r--r--. 1 ntp  ntp   16910 Mar 22 00:59 peerstats.log.20200321
      -rw-r--r--. 2 ntp  ntp    3549 Mar 22 07:14 peerstats.log.20200322
      -rw-r--r--. 2 ntp  ntp     347 Mar 22 08:35 sysstats.log
      -rw-r--r--. 1 ntp  ntp     980 Mar 21 00:46 sysstats.log.20200320
      -rw-r--r--. 1 ntp  ntp    1041 Mar 22 00:35 sysstats.log.20200321
      -rw-r--r--. 2 ntp  ntp     347 Mar 22 08:35 sysstats.log.20200322

## Export Logs to ELK

### Deploy Filebeat

Install filebeat, follow this procedure

Configure filebeat

	vim /etc/filebeat/filebeat.yml

	filebeat.inputs:
	- type: log
	  enabled: true
	  paths:
	    - /var/log/ntpstats/iptables.log
	  fields:
	    logtype: iptables

	- type: log
	  enabled: true
	  paths:
	    - /var/log/ntpstats/peerstats.log
	  fields:
	    logtype: peerstats

	- type: log
	  enabled: true
	  paths:
	    - /var/log/ntpstats/sysstats.log
	  fields:
	    logtype: sysstats
    	
	output.logstash:
  		hosts: ["<ip_logstash>:5044"]
	
Start filebeat

	systemctl start filebeat
	
### Configure Logstash

	touch /etc/logstash/conf.d/00-beats.conf
	vim /etc/logstash/conf.d/00-beats.conf

	input {
	  beats {
	    port => 5044
	  }
	}

	filter {
	  if( [fields][logtype] == "iptables"){
	    dissect {
	      mapping => { "message" => "%{} %{} %{} %{} kernel: [NTP] IN=%{} OUT=%{} MAC=%{} SRC=%{ntp_clientip} %{}" }
	    }
	    if([ntp_clientip] == "<ip1_ntp_peer") {
		drop {}
	    }
	  }
	 if( [fields][logtype] == "peerstats"){
	    dissect {
	      mapping => { "message" => "%{datemjd} %{timepast_midnight} %{src_addr} %{status_word} %{clock_offset} %{roundtrip_delay} %{dispersion} %{rms_jitter}" }
	    }
	   mutate {
		convert => {
		  "clock_offset" => "float"
		  "roundtrip_delay" => "float"
		  "dispersion" => "float"
		  "rms_jitter" => "float"
	       }
	    }
	  }

	 if( [fields][logtype] == "sysstats"){
	    dissect {
	      mapping => { "message" => "%{date_mjd} %{timepast_midnight} %{timesince_reset} %{pkts_received} %{pkts_generated} %{pkts_ntpv4} %{pkts_ntpv3} %{pkts_denied} %{pkts_bad} %{bad_auth} %{declined} %{rate_exceeded} %{pkts_kiss}" }
	    }
	     mutate {
		convert => {
		    "pkts_received" => "integer"
		    "pkts_generated" => "integer"
		    "pkts_ntpv4" => "integer"
		    "pkts_ntpv3" => "integer"
		    "pkts_denied" => "integer"
		    "pkts_bad" => "integer"
		    "bad_auth" => "integer"
		    "declined" => "integer"
		    "rate_exceeded" => "integer"
		    "pkts_kiss" => "integer"
		}
	    }
	  }
	}

	output {
	  elasticsearch {
	    hosts => ["http://localhost:9200"]
	    index => "%{[@metadata][beat]}-%{[@metadata][version]}"
	  }
	}

## Examples of dashboard on Kibana

![ELK dashboard image](/static/images/0002/ntp-elk-dashboard.png)

![ELK dashboard image](/static/images/0002/ntp-elk-dashboard-offset.png)