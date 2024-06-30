---
date: 2024-06-30
layout: post
title: Armbian Hostapd with bridge network sharing same IP range as host
description: Armbian Hostapd with bridge network sharing same IP range as host.
categories:
- Blog
tags:
- Armbian Linux

---

{:toc}

# User netplan to set ip
add the following lines to  /etc/netplan/armbian-default.yaml
```
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enx4a09fe05257f:
      dhcp4: no
      addresses: [192.168.3.1/24]
    end0:
      dhcp4: yes  
```

# Disable system-resolved
add the following line to /etc/systemd/resolved.conf 
```
DNSStubListener=no
```

# 配置lan口
add the following lines to /etc/dnsmasq.conf
```
port=53 #启用局域网DNS
resolv-file=/etc/resolv.dnsmasq.conf
server=192.168.1.1
listen-address=192.168.3.1,127.0.0.1
dhcp-range=192.168.3.50,192.168.3.250,24
dhcp-option=3,192.168.3.1
dhcp-option=6,192.168.3.1,119.29.29.29
cache-size=1500
min-cache-ttl=1200
```
# Setup iptables
```
iptables -t nat -I POSTROUTING -j MASQUERADE
```

# brctl to list all bridge 

sudo brctl show

```
# 配置 enx4a09fe05257f 接口为手动模式
nmcli connection add type ethernet ifname enx4a09fe05257f con-name enx4a09fe05257f

# 创建桥接接口 br1，并配置其静态 IP 和桥接参数
nmcli connection add type bridge ifname br1 con-name br1
nmcli connection modify br1 ipv4.method manual ipv4.addresses 192.168.5.1/24
nmcli connection modify br1 bridge.stp on
nmcli connection modify br1 bridge.forward-delay 2

# 将 enx4a09fe05257f 添加到桥接 br1
nmcli connection add type bridge-slave ifname enx4a09fe05257f master br1

# 启用 br1 接口
nmcli connection up br1
```
