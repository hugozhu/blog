---
date: 2016-04-06
layout: post
title:  用iptables搭建稳定的加速代理
description: use iptables to set up smart route
categories:
- Blog
tags:
- Raspberry Pi
- iptables


---

{:toc}

# 设置步骤

在阿里云中国和美国都购买一个VPS，用同样的操作系统，都安装好 `shadowsocks-libev` ( https://github.com/shadowsocks/shadowsocks-libev )

## 编辑配置文件 `config.json` 如下：

```
{
    "server":"<your_ip_address>",
    "local_address":"0.0.0.0",
    "server_port":10080,
    "local_port":1080,
    "password":"password",
    "method":"bf-cfb",
    "timeout":600
}
```

## 在美国的服务器上执行：

```
nohup ss-server config.json &
```

## 在中国的服务器上执行：

```
nohup ss-redir config.json &
```

## 在中国的服务器上安装好L2TP服务，

`/etc/xl2tpd/xl2tpd.conf` 里设置好vpn ip段

```
[global]
ipsec saref = yes
listen-addr = <外网IP>
[lns default]
ip range = 192.168.1.10-192.168.1.20
local ip = 192.168.1.1
;require chap = yes
refuse chap = yes
refuse pap = yes
require authentication = yes
ppp debug = yes
pppoptfile = /etc/ppp/xl2tpd-options
length bit = yes
```

/etc/ppp/xl2tpd-options 里设置通过vpn接入进来的设备DNS

```
asyncmap 0
auth
crtscts
lock
hide-password
modem
mru 1280
netmask 255.255.255.0
mtu 1280
name l2tpd
proxyarp
lcp-echo-interval 30
lcp-echo-failure 4
noipx
ms-dns 192.168.1.1
ms-dns 8.8.8.8
```

## 设置iptables

```
*nat
-A PREROUTING -s 192.168.1.0/24 -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 1080
-A PREROUTING -s 192.168.1.0/24 -p tcp -m tcp --dport 443 -j REDIRECT --to-ports 1080
-A POSTROUTING -s 192.168.1.0/24 -o eth1 -j MASQUERADE
-A OUTPUT -p tcp -m tcp --dport 53 -j REDIRECT --to-ports 1080
COMMIT
```

## 设置pdns

```
global {
    perm_cache=4096;
    cache_dir="/var/cache/pdnsd";
    max_ttl=604800;
    query_method=tcp_only;
    paranoid=on;
    server_port=53;
    server_ip="127.0.0.1";
    min_ttl=1d;
    max_ttl=1w;   
}

server {
    label="GoogleDNS";
    ip=8.8.8.8;
	port = 53;
	proxy_only=on;
    timeout=30;
    interval=30;
    uptest=none;
	preset=on;
	root_server=off;
}

source {
	owner=localhost;
	file="/etc/hosts";
}

rr {
	name=localhost;
	reverse=on;
	a=127.0.0.1;
	owner=localhost;
	soa=localhost,root.localhost,42,86400,900,86400,86400;
}
```