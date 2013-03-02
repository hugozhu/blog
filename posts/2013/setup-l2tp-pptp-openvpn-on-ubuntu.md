---
date: 2013-03-01
layout: post
title: 在Ubuntu上配置L2TP，PPTP和OpenVPN服务
categories:
- Blog
tags:
- VPN

---

## Overview

MacOS, Windows, iOS都内置支持PPTP，L2TP；OpenVPN需要安装客户端，手机上一般不支持。

先打开内核的IP转发，修改 /etc/sysctl.conf

    net.ipv4.ip_forward=1
    
执行下面命令以生效

    sudo sysctl -p
   
## PPTP

安装pptpd

    apt-get install pptpd
    

编辑 /etc/pptpd.conf，下面两行取消注释
    
    localip 192.168.0.1
    remoteip 192.168.0.234-238,192.168.0.245
    
这行注释掉

    #logwtmp 
    
从文件 /etc/pptpd.conf 中找到配置选项文件，如下为：/etc/ppp/pptpd-options

    grep options /etc/pptpd.conf
    #       Specifies the location of the PPP options file.
    #       By default PPP looks in '/etc/ppp/options'
    option /etc/ppp/pptpd-options
    #       option in the pppd options file, or run bcrelay.
    
编辑 /etc/ppp/pptpd-options，增加以下内容，最后两项为推给VPN客户端的DNS服务器IP
    
    mtu 1492
    name pptpd
    refuse-pap
    refuse-chap
    refuse-mschap
    require-mschap-v2
    require-mppe-128
    proxyarp
    lock
    nobsdcomp
    novj
    novjccomp
    nologfd 
    ms-dns 8.8.8.8
    ms-dns 8.8.4.4
   
修改 /etc/ppp/chap-secrets， 增加一个VPN用户: foo ，密码设置为: bar

    # Secrets for authentication using CHAP
    # client        server  secret                  IP addresses     
    foo    pptpd   bar   *
    
修改 iptable，注意eth0可能要修改成实际的网络接口名（用 ifconfig 可以列出）

    iptables -F
    iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
    iptables -A INPUT -i lo -j ACCEPT
    iptables -A INPUT -i tap+ -j ACCEPT
    iptables -A INPUT -i tun+ -j ACCEPT
    iptables -A FORWARD -i tap+ -j ACCEPT
    iptables -A FORWARD -i tun+ -j ACCEPT
    iptables -P FORWARD ACCEPT    
    
## L2TP Over IPSec

假设你的服务器IP是：**1.2.3.4**

首先更新一下源

    sudo apt-get update
    
安装openswan

    sudo apt-get install openswan

    sudo cp /etc/ipsec.d/examples/l2tp-psk.conf /etc/ipsec.d/l2tp-psk.conf
    
修改文件 /etc/ipsec.d/l2tp-psk.conf

    left=1.2.3.4 #机器的外部IP
    leftnexthop=1.2.3.1 #机器的Gateway
    
修改 /etc/ipsec.conf，在文件最后增加：

    include /etc/ipsec.d/l2tp-psk.conf

修改 /etc/ipsec.secrets
    
    1.2.3.4 %any: "yourSharedPSK!"

安装 xl2tpd
   
    apt-get install xl2tpd
    
修改 /etc/xl2tpd/xl2tpd.conf , 其中1.2.3.4改成服务器的外部IP
   
    [global]
    ipsec saref = yes
    listen-addr = 1.2.3.4
    
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
    
配置 /etc/ppp/xl2tpd-options

    cp /etc/ppp/options /etc/ppp/xl2tpd-options
    
修改 /etc/ppp/xl2tpd-options

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
    ms-dns 8.8.8.8
    ms-dns 8.8.4.4

修改 /etc/ppp/chap-secrets， 增加一个VPN用户: foo ，密码设置为: bar

    # Secrets for authentication using CHAP
    # client        server  secret                  IP addresses     
    foo    l2tpd   bar   *


修改 iptable，注意eth0可能要修改成实际的网络接口名（用 ifconfig 可以列出）, 其中1.2.3.4改成服务器的外部IP

    iptables -F
    iptables -A INPUT -p 50 -j ACCEPT
    iptables -A INPUT -p udp -d 1.2.3.4 --dport 500 -j ACCEPT
    iptables -A INPUT -p udp -d 1.2.3.4 --dport 4500 -j ACCEPT


## OpenVPN
    
安装Openvpn
    
    sudo apt-get install openvpn    
    
    cp -r /usr/share/doc/openvpn/examples/easy-rsa/ /etc/openvpn/

生成CA证书

    cd /etc/openvpn/easy-rsa/2.0
    source vars
    ./clean-all
    ./build-ca
    ./build-key-server server
    ./build-key client
    ./build-dh
    
编辑/etc/openvpn/server.conf
    
    local 116.251.211.71    
    port 56788    
    proto tcp    
    dev tun
    ca /etc/openvpn/easy-rsa/2.0/keys/ca.crt
    cert /etc/openvpn/easy-rsa/2.0/keys/server.crt
    key /etc/openvpn/easy-rsa/2.0/keys/server.key      
    dh  /etc/openvpn/easy-rsa/2.0/keys/dh1024.pem    
    server 10.8.0.0 255.255.255.0    
    ifconfig-pool-persist ipp.txt
    push "redirect-gateway def1"    
    push "dhcp-option DNS 8.8.8.8"
    push "dhcp-option DNS 8.8.4.4"    
    client-to-client
    keepalive 10 120
    comp-lzo    
    max-clients 50    
    user nobody
    group nogroup    
    persist-key
    persist-tun    
    status openvpn-status.log    
    log-append  openvpn.log    
    verb 3    
    mute 20    

设置iptable，其中1.2.3.4改成服务器的外部IP
    
    iptables -F
    iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o venet0 -j MASQUERADE
      
    iptables -A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
    iptables -t nat -A PREROUTING -p udp -m udp --dport 53 -j DNAT --to-destination 8.8.8.8
     
    iptables -A INPUT -p udp --dport 1194 -j ACCEPT
    iptables -A INPUT -s 10.8.0.0/24 -p all -j ACCEPT
    iptables -A FORWARD -d 10.8.0.0/24 -j ACCEPT
     
    iptables -A INPUT -i tun+ -j ACCEPT
    iptables -A FORWARD -i tun+ -j ACCEPT
    iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j SNAT --to-source 1.2.3.4
      