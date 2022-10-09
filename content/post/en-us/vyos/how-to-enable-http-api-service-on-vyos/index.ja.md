---
title: "VyOS で API サービスを有効にする方法"
date: 2022-04-26T00:17:15+08:00
lastmode: 2022-04-26T00:17:15+08:00
draft: false

tags: [ "vyos", "api" ]
categories: ["network"]
layout: "page"
slug: "How to Enable Http Api Service on VyOS"
reward: true
mathjax: true
image: image: https://picsum.photos/800/600.webp?random=71ff97d5
---


# VyOS で API サービスを有効にする方法

最近になって、ネットワーク ステータスを自動検出し、いくつかの設定を自動的に変更する vyos シェル スクリプトを作成しました。 途中で、直接ロードするための新しい構成があり、構成モードをより長く保持します。 これは賢明な方法ではありません。

別のより良いオプションとして、API を使用して vyos の設定を変更できるとしたらどうでしょうか?

## API https サービスをローカルで有効にする
### ソケットモードで apt サービスを有効にする
```bash
set service https api socket 
```

### テスト設定

```bash
curl --unix-socket /run/api.sock -X POST -Fkey=qwerty -Fdata='{"op": "showConfig", "path": []}' http://localhost/retrieve
```

## Let's暗号化フリーsslでapi httpsサービスを有効にする

```bash
#!/bin/vbash
source /opt/vyatta/etc/functions/script-template
configure

# 変数の定義
ID="my_id"
APIKEY="12345"
APIPORT="1234"
VHOST="myvps"
FULL_FQDN="xxx.xxx.xxx"
LISTEN_ADDRESS="IP ADDRESS"
EMAIL="my@emal.com"

set service https api key id $ID key $APIKEY
set service https certificates certbot domain-name $FULL_FQDN
set service https certificates certbot email $EMAIL
set service https api strict
set service https virtual-host $VHOST listen-address $LISTEN_ADDRESS
set service https virtual-host $VHOST listen-port $APIPORT
set service https virtual-host $VHOST server-name $FULL_FQDN
commit
save
exit
```




