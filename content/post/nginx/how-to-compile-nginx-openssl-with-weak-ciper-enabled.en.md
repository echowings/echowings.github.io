---
title: "How to build nginx docker image with openssl weak "
date: 2023-02-15T08:46:43+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=0217b741
categories:
  - nginx
tags:
  - nginx
  - openssl
  - weak
  - docker
slug:
  - nginx
layout: 
  - post


license: CC BY-NC-ND
---



## Dockerfile

```dockerfile
FROM debian:bullseye-slim

LABEL maintainer="Steve Dong <dongjunbo@gmail.com>"

ENV NGINX_VERSION 1.23.3
ENV OPENSSL_VERSION 1.1.1t

RUN set -x \
	&& apt-get update \
	&& apt-get install --no-install-recommends --no-install-suggests -y \
		gnupg1 apt-transport-https ca-certificates libpcre3 zlib1g \
		build-essential wget libpcre3-dev zlib1g-dev \
	&& wget -O - https://www.openssl.org/source/openssl-"$OPENSSL_VERSION".tar.gz | tar -xzf - -C /tmp  \
        && wget -O - http://nginx.org/download/nginx-"$NGINX_VERSION".tar.gz | tar -xzf - -C /tmp \
        && cd /tmp/nginx-"$NGINX_VERSION" \
        && ./configure \
		--prefix=/etc/nginx \
		--sbin-path=/usr/sbin/nginx \
		--modules-path=/usr/lib64/nginx/modules \
		--conf-path=/etc/nginx/nginx.conf \
		--error-log-path=/var/log/nginx/error.log \
		--http-log-path=/var/log/nginx/access.log \
		--pid-path=/var/run/nginx.pid \
		--lock-path=/var/run/nginx.lock \
		--http-client-body-temp-path=/var/cache/nginx/client_temp \
		--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
		--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
		--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
		--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
		--user=nginx \
		--group=nginx \
		--with-file-aio \
		--with-threads \
		--with-ipv6 \
		--with-http_addition_module \
		--with-http_auth_request_module \
		--with-http_dav_module \
		--with-http_flv_module \
		--with-http_gunzip_module \
		--with-http_gzip_static_module \
		--with-http_mp4_module \
		--with-http_random_index_module \
		--with-http_realip_module \
		--with-http_secure_link_module \
		--with-http_slice_module \
		--with-http_ssl_module \
		--with-http_stub_status_module \
		--with-http_sub_module \
		--with-http_v2_module \
		--with-mail \
		--with-mail_ssl_module \
		--with-stream \
		--with-stream_ssl_module \
		--with-openssl=/tmp/openssl-"$OPENSSL_VERSION" --with-openssl-opt=enable-weak-ssl-ciphers \
		--with-cc-opt='-O2 -g -pipe -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic' \
	&& \
	make \
        && cd /tmp/nginx-"$NGINX_VERSION" \
        && groupadd -r nginx \
        && useradd -r -g nginx nginx \
	&& make install \
	&& mkdir /var/cache/nginx \
        && chown nginx:root /var/cache/nginx \
        && chown -R nginx:root /var/log/nginx \
        && apt-get purge -y build-essential wget libpcre3-dev zlib1g-dev \
        && ln -sf /dev/stdout /var/log/nginx/access.log \
        && ln -sf /dev/stderr /var/log/nginx/error.log \
        && rm -rf /tmp/*.tar.gz /tmp/nginx-* /tmp/openssl-* \
        && apt-get -f -y autoremove

EXPOSE 80 443

STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]
```

