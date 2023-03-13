---
title: "How to Deploy Mosdns and adguard With Docker-Compose"
date: 2023-02-25T23:04:40+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=aca77a1c
categories:
  - networking
tags:
  - mosdns
  - networking
  - docker
  - docker-compose
slug:
  - mosdns
layout: 
  - post


license: CC BY-NC-ND
---

# How to deploy mosdns and adguard with docker-compose
## Configure mosdns

### Create folder and download dns data to ./date folder
```bash
mkdir -p /data/mosdns-dockercompose && \
cd /data/mosdns-dockercompose && \
rm -rf ./data && \ 
mkdir ./data && \
curl https://raw.githubusercontent.com/Loyalsoldier/v2ray-rules-dat/release/direct-list.txt > ./data/direct-list.txt && \
curl https://raw.githubusercontent.com/Loyalsoldier/v2ray-rules-dat/release/apple-cn.txt    > ./data/apple-cn.txt && \
curl https://raw.githubusercontent.com/Loyalsoldier/v2ray-rules-dat/release/google-cn.txt   > ./data/google-cn.txt && \
curl https://raw.githubusercontent.com/Loyalsoldier/v2ray-rules-dat/release/proxy-list.txt  > ./data/proxy-list.txt && \
curl https://raw.githubusercontent.com/Loyalsoldier/v2ray-rules-dat/release/gfw.txt              > ./data/gfw.txt && \
curl https://raw.githubusercontent.com/Hackl0us/GeoIP2-CN/release/CN-ip-cidr.txt                 > ./data/CN-ip-cidr.txt && \
touch ./data/force-nocn.txt && \
touch ./data/force-cn.txt && \
touch ./data/hosts 
```

### Create `./data/config.yaml`
path: `./data/config.yaml`

```yaml
log:
    level: error
    production: true

# API 入口设置
api:
  http: "0.0.0.0:9080" # 在该地址启动 api 接口。

# 从其他配置文件载入 plugins 插件设置。
# include 的插件会比本配置文件中的插件先初始化。
include: []

plugins:
  - tag: "geosite-cn"
    type: domain_set
    args:
      files:
        - "./direct-list.txt"
        - "./apple-cn.txt"
        - "./google-cn.txt"

  - tag: "geosite-nocn"
    type: domain_set
    args:
      files:
        - "./proxy-list.txt"
        - "./gfw.txt"

  - tag: "geoip-cn"
    type: ip_set
    args:
      files: "./CN-ip-cidr.txt"

  - tag: "force-cn"
    type: domain_set
    args:
      files: "./force-cn.txt"

  - tag: "force-nocn"
    type: domain_set
    args:
      files: "./force-nocn.txt"

  - tag: "hosts"
    type: hosts
    args:
      files: "./hosts.txt"

  - tag: "cache"
    type: "cache"
    args:
      size: 1024
      lazy_cache_ttl: 0
      dump_file: ./cache.dump
      dump_interval: 600

  # 转发至本地服务器的插件
  - tag: forward_local
    type: forward
    args:
      concurrent: 3
      upstreams:
        - addr: "udp://114.114.114.114"
        - addr: "udp://223.5.5.5"
        - addr: "udp://223.6.6.6"

  # 转发至远程服务器的插件
  - tag: forward_remote
    type: forward
    args:
      concurrent: 3
      upstreams:
        - addr: "udp://1.1.1.1"
        - addr: "udp://8.8.8.8"
        - addr: "udp://8.8.4.4"

  - tag: "primary_forward"
    type: sequence
    args:
      - exec: $forward_local
      - exec: ttl 60-3600
      - matches:
        - "!resp_ip $geoip-cn"
        - "has_resp"
        exec: drop_resp

  - tag: "secondary_forward"
    type: sequence
    args:
      - exec: prefer_ipv4
      - exec: $forward_remote
      - matches:
        - rcode 2
        exec: $forward_local
      - exec: ttl 300-3600


  - tag: "final_forward"
    type: fallback
    args:
      primary: primary_forward
      secondary: secondary_forward
      threshold: 150
      always_standby: true

  - tag: main_sequence
    type: sequence
    args:
      - exec: $hosts
      - exec: query_summary hosts
      - matches: has_wanted_ans
        exec: accept

      - exec: $cache
      - exec: query_summary cache
      - matches: has_wanted_ans
        exec: accept

      - exec: query_summary qtype65
      - matches:
        - qtype 65
#         exec: black_hole 127.0.0.1 ::1 0.0.0.0
        exec: reject 0

      - matches:
        - qname $geosite-cn
        exec: $forward_local
      - exec: query_summary geosite-cn
      - matches: has_wanted_ans
        exec: accept

      - matches:
        - qname $force-cn
        exec: $forward_local
      - exec: query_summary force-cn
      - matches: has_wanted_ans
        exec: accept

      - matches:
        - qname $geosite-nocn
        exec: $forward_remote
      - exec: query_summary geosite-nocn
      - matches: has_wanted_ans
        exec: accept

      - matches:
        - qname $force-nocn
        exec: $forward_remote
      - exec: query_summary force-nocn
      - matches: has_wanted_ans
        exec: accept

      - exec: $final_forward

  - tag: "udp_server"
    type: "udp_server"
    args:
      entry: main_sequence
      listen: 0.0.0.0:5353

  - tag: "tcp_server"
    type: "tcp_server"
    args:
      entry: main_sequence
      listen: 0.0.0.0:5353
```

### Create `docker-compose.yaml`
**Path:** `./docker-compose.yaml`

```yaml
version: "3.8"
services:
  mosdns:
    container_name: mosdns
    image: irinesistiana/mosdns:latest
    restart: always
    volumes:
      - ./mosdns-data:/etc/mosdns
    network_mode: "host"
```

## Deploy ADguard
### Create ADGUARD configure file
```bash
mkdir conf.d
touch conf.d/AdGuardHome.yaml
```
conf.d/AdGuardHome.yaml

|USERNAME|PASSWORD|
|---|---|
|admin|xA123456|


```yaml
bind_host: 0.0.0.0
bind_port: 3000
beta_bind_port: 0
users:
  - name: admin
    password: $2y$05$5tqrLmdpGL8pvxt2ZDgokOqe/qioxAK3FwT46afRK9sIiXS1cRAnO
auth_attempts: 5
block_auth_min: 15
http_proxy: ""
language: ""
debug_pprof: false
web_session_ttl: 720
dns:
  bind_hosts:
    - 0.0.0.0
  port: 53
  statistics_interval: 1
  querylog_enabled: true
  querylog_file_enabled: true
  querylog_interval: 24h
  querylog_size_memory: 1000
  anonymize_client_ip: false
  protection_enabled: true
  blocking_mode: default
  blocking_ipv4: ""
  blocking_ipv6: ""
  blocked_response_ttl: 10
  parental_block_host: family-block.dns.adguard.com
  safebrowsing_block_host: standard-block.dns.adguard.com
  ratelimit: 0
  ratelimit_whitelist: []
  refuse_any: true
  upstream_dns:
    - udp://127.0.0.1:5353
  upstream_dns_file: ""
  bootstrap_dns:
    - 8.8.8.8
    - 8.8.4.4
    - 9.9.9.11
    - 149.112.112.11
  all_servers: true
  fastest_addr: false
  fastest_timeout: 1s
  allowed_clients:
    - 0.0.0.0/0
  disallowed_clients: []
  blocked_hosts:
    - version.bind
    - id.server
    - hostname.bind
  trusted_proxies:
    - 0.0.0.0/0
    - 127.0.0.0/8
    - ::1/128
  cache_size: 0
  cache_ttl_min: 0
  cache_ttl_max: 0
  cache_optimistic: false
  bogus_nxdomain: []
  aaaa_disabled: false
  enable_dnssec: false
  edns_client_subnet: false
  max_goroutines: 300
  handle_ddr: true
  ipset: []
  filtering_enabled: true
  filters_update_interval: 24
  parental_enabled: false
  safesearch_enabled: true
  safebrowsing_enabled: true
  safebrowsing_cache_size: 1048576
  safesearch_cache_size: 1048576
  parental_cache_size: 1048576
  cache_time: 30
  rewrites: []
  blocked_services: []
  upstream_timeout: 10s
  private_networks: []
  use_private_ptr_resolvers: true
  local_ptr_upstreams: []
tls:
  enabled: false
  server_name: ""
  force_https: false
  port_https: 443
  port_dns_over_tls: 853
  port_dns_over_quic: 853
  port_dnscrypt: 0
  dnscrypt_config_file: ""
  allow_unencrypted_doh: false
  strict_sni_check: false
  certificate_chain: ""
  private_key: ""
  certificate_path: ""
  private_key_path: ""
filters:
  - enabled: false
    url: https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
    name: AdGuard DNS filter
    id: 1
  - enabled: false
    url: https://adaway.org/hosts.txt
    name: AdAway Default Blocklist
    id: 2
  - enabled: false
    url: https://raw.githubusercontent.com/privacy-protection-tools/anti-AD/master/anti-ad-easylist.txt
    name: anti-ad-easylist
    id: 1662217242
  - enabled: true
    url: https://gitee.com/halflife/list/raw/master/ad.txt
    name: HalfLife
    id: 1662217243
  - enabled: true
    url: https://gitee.com/xinggsf/Adblock-Rule/raw/master/rule.txt
    name: xinggsf，乘风广告过滤规则
    id: 1662217244
  - enabled: true
    url: https://easylist-downloads.adblockplus.org/easyprivacy.txt
    name: EasyPrivacy
    id: 1662217245
  - enabled: false
    url: https://www.i-dont-care-about-cookies.eu/abp/
    name: I don’t care about cookies
    id: 1662217246
  - enabled: true
    url: https://anti-ad.net/adguard.txt
    name: anti-AD Filters
    id: 1662217247
  - enabled: true
    url: https://gist.githubusercontent.com/Ewpratten/a25ae63a7200c02c850fede2f32453cf/raw/b9318009399b99e822515d388b8458557d828c37/hosts-yt-ads
    name: YouTube-去广告
    id: 1662217248
  - enabled: true
    url: https://anti-ad.net/easylist.txt
    name: easylist.txt
    id: 1662225908
  - enabled: false
    url: https://raw.githubusercontent.com/Goooler/1024_hosts/master/hosts
    name: 1024网站及澳门皇家赌场及恶意广告主机列表
    id: 1662918853
  - enabled: true
    url: https://gitee.com/xinggsf/Adblock-Rule/raw/master/mv.txt
    name: 乘风视频规则（周更），可过滤爱优腾三大视频网站的片头秒跳广告
    id: 1662918854
  - enabled: false
    url: https://cdn1.tianli0.top/gh/zqzess/rule_for_quantumultX@master/Loon/Plugin/AdBlock.plugin
    name: 圈X规则 zqzess/rule_for_quantumultX：常见国内app去广告
    id: 1662918855
  - enabled: true
    url: https://raw.githubusercontent.com/v2ray/domain-list-community/master/data/google-ads
    name: google ads
    id: 1662918856
  - enabled: true
    url: https://raw.githubusercontent.com/alexsannikov/adguardhome-filters/master/porn.txt
    name: porn
    id: 1677740002
  - enabled: false
    url: https://abp.oisd.nl/basic/
    name: OISD Blocklist Basic
    id: 1678418114
  - enabled: false
    url: https://raw.githubusercontent.com/durablenapkin/scamblocklist/master/adguard.txt
    name: Scam Blocklist by DurableNapkin
    id: 1678418115
  - enabled: false
    url: https://raw.githubusercontent.com/hoshsadiq/adblock-nocoin-list/master/hosts.txt
    name: NoCoin Filter List
    id: 1678418116
  - enabled: false
    url: https://raw.githubusercontent.com/DandelionSprout/adfilt/master/Alternate%20versions%20Anti-Malware%20List/AntiMalwareAdGuardHome.txt
    name: Dandelion Sprout's Anti-Malware List
    id: 1678418117
  - enabled: false
    url: https://malware-filter.gitlab.io/malware-filter/urlhaus-filter-agh-online.txt
    name: Online Malicious URL Blocklist
    id: 1678418118
  - enabled: false
    url: https://raw.githubusercontent.com/mitchellkrogza/The-Big-List-of-Hacked-Malware-Web-Sites/master/hosts
    name: The Big List of Hacked Malware Web Sites
    id: 1678418119
  - enabled: true
    url: https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/spy.txt
    name: WindowsSpyBlocker - Hosts spy rules
    id: 1678418120
  - enabled: true
    url: https://big.oisd.nl/
    name: oisd-big
    id: 1678418121
  - enabled: true
    url: https://someonewhocares.org/hosts/zero/hosts
    name: Dan Pollock's List
    id: 1678636426
  - enabled: true
    url: https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/SmartTV-AGH.txt
    name: Perflyst and Dandelion Sprout's Smart-TV Blocklist
    id: 1678636427
  - enabled: true
    url: https://pgl.yoyo.org/adservers/serverlist.php?hostformat=adblockplus&showintro=1&mimetype=plaintext
    name: Peter Lowe's List
    id: 1678636428
  - enabled: true
    url: https://raw.githubusercontent.com/DandelionSprout/adfilt/master/GameConsoleAdblockList.txt
    name: Game Console Adblock List
    id: 1678636429
whitelist_filters: []
user_rules:
  - ""
dhcp:
  enabled: false
  interface_name: ""
  local_domain_name: lan
  dhcpv4:
    gateway_ip: ""
    subnet_mask: ""
    range_start: ""
    range_end: ""
    lease_duration: 86400
    icmp_timeout_msec: 1000
    options: []
  dhcpv6:
    range_start: ""
    lease_duration: 86400
    ra_slaac_only: false
    ra_allow_slaac: false
clients:
  runtime_sources:
    whois: true
    arp: true
    rdns: true
    dhcp: true
    hosts: true
  persistent: []
log_file: ""
log_max_backups: 0
log_max_size: 100
log_max_age: 3
log_compress: false
log_localtime: false
verbose: false
os:
  group: ""
  user: ""
  rlimit_nofile: 0
schema_version: 14
```

### Create `docker-compose.yaml`

```yaml
version: '3.9'
services:
  adguardhome:
    container_name: adguardhome
    image: adguard/adguardhome
    network_mode: "host"
    ports:
      - 3000:3000
    volumes:
      - ./data:/opt/adguardhome/work/data
      - ./conf.d:/opt/adguardhome/conf
      - /etc/localtime:/etc/localtime:ro
    restart: always
```




