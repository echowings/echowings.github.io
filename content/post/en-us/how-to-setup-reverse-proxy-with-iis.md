---
title: "How to Setup Reverse Proxy With Iis"
date: 2018-11-06T23:26:18+08:00
tags: ["windows","iis","reverse proxy"]
categories: [ "reverse proxy" ]
author: "steve dong"
draft: false
reward: true
---

# How to Setup Reverse proxy with IIS
I testing it on Server 2016 with IIS 10

## Install IIS,URL _REWRITE, ARR

  * Install IIS.
  * Download and install [URL Rewrite Module 2.1](https://www.iis.net/downloads/microsoft/url-rewrite)
  * Download and install [ARR-Application-request-routing](https://www.iis.net/downloads/microsoft/application-request-routing)

## Setting Reverse Proxy
  * Select Root level in IIS management
  * Click ‘Application Request Routing Cache’,then in the right side, click “proxy | Server proxy Settings | Check on “Enable proxy” | save config.
  * Select website in IIS.
  * Click URL Rewrite
  * Add a black Rule

 		* Match URL:

|Requested URL	|Using	|Pattern|
|---|---|---|
|Matches the pattern	|Regular Expressions|	`^(.*)` |

		* Conditions:

|Type	|Value|
|---|---|
|Logical grouping	|Match ALL|
|input	|`{HTTP_HOST}`|
|Type	|Matches the Pattern|
|Patern	| `^xa-sys-svr:8080$` |

		* Server Variables:

|Type	|Value|
|---|---|
|Action Type|	Rewrite|
|Rewrite URL	|`http://www.cnblog.com/{R:1}`|


## Reference

 * [用IIS配置反向代理](https://my.oschina.net/tanyixiu/blog/123832)
