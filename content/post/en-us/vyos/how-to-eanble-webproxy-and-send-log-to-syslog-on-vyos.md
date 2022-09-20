---
title: "How to Enable Webproxy and Send Log to Syslog on Vyos"
date: 2019-11-21T20:57:57+08:00
lastmode: 2019-11-21T20:57:57+08:00
draft: true
tags: [ "vyos", "squid","syslog", "graylog" ]
categories: ["networking"]
reward: true
mathjax: true
---

# How to enable webproxy and send log to syslog on vyos


## Enable webproxy on vyos


```bash
config
# set cache 0
set serivce webproxy cache-size '0'

# set webproxy listen on <ip address of vyos>  and disable transparent mode
set service webproxy listen-address 10.32.0.6 disable-transparent
```

## Change webproxy log from logfile to syslog

```bash
sudo vi /opt/vyatta/sbin/vyatta-update-webproxy.pl

#chang 
my $squid_log       = '/var/log/squid3/access.log';

#to
#my $squid_log       = '/var/log/squid3/access.log';
my $squid_log       = 'syslog:local2';

```


## Swell


- [Solved: Webproxy (squid) access log - write to remote syslog only](https://community.ui.com/questions/Solved-Webproxy-squid-access-log-write-to-remote-syslog-only/74a55d86-f7a0-4013-82a3-6cc4e7862e45)
