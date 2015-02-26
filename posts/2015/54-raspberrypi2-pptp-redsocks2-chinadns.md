---
date: 2015-02-26
layout: post
title: 用树莓派2代打造智能家庭路由
description: Set up your RaspberryPi 2 as a smart router
categories:
- Blog
tags:
- Raspberry Pi


---

{:toc}


家里的宽带上下行都有10Mbps了吧？除了可以BT下载外还能让你的移动设备在外的时候也能科学上网。

区别于在海外架设VPN服务：

	1）国内和大部分国外网站都可以直连而不降低速度；不像海外VPN所有流量（国内和国外网站）都要经过海外服务器，速度有一定的延迟；
	2）这个方案代理可以使用免费的Goagent服务；或低成本的ssh帐号；而租用海外VPS服务器自建服务或购买VPN帐号费用较高些；
	3）利用的是家里的宽带，只有树莓派的硬件成本，没有主机托管成本；

假设家里的路由器IP地址为:**192.168.1.1**，树莓派2的IP地址为:**192.168.1.3**，以下是需要安装和设置步骤。


# PPTP和L2TP VPN Server

首先在树莓派上安装和设置VPN服务器，移动设备就可以通过运营商网络连接回家里的树莓派（iPhone和Android都内置了PPTP和L2TP客户端），这样移动设备将以树莓派为路由访问网站，通过一些设置我们可以让树莓派提供科学上网服务。

关于PPTP和L2TP VPN设置和安装可以参考：http://hugozhu.myalert.info/2013/03/01/setup-l2tp-pptp-openvpn-on-ubuntu.html

但在树莓派上安装L2TP时不能直接`apt-get install openswan`，需要手动下载来安装，原因是因为最新的版本在协议上有些不兼容：

```
wget http://snapshot.raspbian.org/201403301125/raspbian/pool/main/o/openswan/openswan_2.6.37-3_armhf.deb
sudo dpkg -i openswan_2.6.37-3_armhf.deb
```

假设VPN服务端的local ip我们设置为`192.168.3.1`，PPTP客户端IP分配区间为：`192.168.3.200~192.168.3.210`，L2TP 客户端IP分配区间为：`192.168.3.100~192.168.3.110`，我们可以通过`iptables`对IP来源为192.168.3.0/24网段的流量做特殊的处理以达到科学上网的目的。

完成这一步后，需要在路由器上设置端口转发，使得使用运营商网络如移动4G的手机可以通过PPTP或L2TP连到树莓派上。


PPTP需要设置的端口转发 - tcp: 1723

L2TP需要设置的端口转发 - tcp: 50, udp: 500,4500,1701

PPTP拨号速度比较快，但是不安全；L2TP有加密，相对安全。


# Redsocks2

[redsocks2](http://github.com/hugozhu/redsocks)是一个透明TCP代理，其实现使用了libevent库，性能较好，其最大的特点是如果目标IP可以直连则不会转发流量给加密代理，如果IP不能直连（通过连接超时判断）则会将流量转发给加密代理。这样可以将最少的流量转发到代理上，访问一般的国外网站如yahoo.com也不会经过代理而减速，在配置方面则做到了零配置，不需要手工维护网站名单。代理也能支持很多中类型，如socks5, shadowsocks, goagent, http-proxy等，redsocks2安装和配置可以见链接： http://github.com/hugozhu/redsocks

这里我们假设redsocks2的端口使用**12345**


# iptables

使用iptables我们可以将某些来源的流量转发到本地的某个端口

```
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F
sudo iptables -t nat -X
sudo iptables -t nat -A PREROUTING -s 192.168.3.0/24 -p tcp --dport 80 -j REDIRECT --to-ports 12345 #转发VPN客户端的HTTP流量到端口12345
sudo iptables -t nat -A PREROUTING -s 192.168.3.0/24 -p tcp --dport 443 -j REDIRECT --to-ports 12345 #转发VPN客户端的HTTPS流量到端口12345
sudo iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -o eth0 -j MASQUERADE #转发VPN客户端的TCP流量到网络出口，并进行IP伪装；如果树莓派使用无线网卡则将eth0改成wlan0
```

# DNS加固

上面的设置我们解决了VPN拨号到树莓派的客户端通过redsocks2透明代理分流为直连或通过加密代理连接和访问目标网站，我们还需要解决一下DNS查询被纂改为不存在的IP地址的问题。

## ChinaDNS
虽然`redsocks2`也具备通过tcp查询DNS而防污染的功能，但实测性能并不怎么好。[ChinaDNS](https://github.com/clowwindy/ChinaDNS)是一个可以防污染的DNS服务器，内存占用小，同时使用114和Google DNS,OpenDNS，AliDNS等，国内网站无延迟；

## dnsmasq
dnsmasq是一个小巧的DNS和DHCP服务器软件，是openwrt的标配，具备dns缓存的功能，建议使用dnsmasq作为解析服务器，ChinaDNS则作为dnsmasq的上游(upstream)服务器，这样搭配较稳定又能缓存解析结果而提高并发性能，dnsmasq安装和配置如下：

安装：
```
sudo apt-get install dnsmasq
```

配置：
```
no-resolv
server=127.0.0.1#1053
```

完成这一步后，我们可以修改pptp和l2tp服务的配置将vpn客户端的DNS服务器设置为`192.168.3.1`


# 总结

通过以上的设置，可以充分发挥树莓派2的4核CPU性能，不仅可以为移动设备提供科学上网服务，也可以为家里的支持PPTP或L2TP VPN的台式机提供同样的服务。如果要进一步折腾，还可以通过增加usb网卡和交换机，让树莓派2作为主路由提供上网服务。