---
date: 2024-06-30
layout: post
title: R2S使用场景：基础系统Armbian，Docker安装openwrt，USB网卡做热点，无线接入通过openwrt做路由
description: Armbian Hostapd with bridge network sharing same IP range as host.
categories:
- Blog
tags:
- Armbian Linux

---

{:toc}

# 背景
Armbian 是一款基于 Debian 或 Ubuntu 的开源操作系统，专门针对嵌入式 ARM 平台进行优化和定制。它可以运行在多种不同的嵌入式设备上，例如树莓派、R2S，R4S，玩客云等等。Armbian 针对不同的嵌入式平台，提供了相应的硬件支持，可以让用户轻松地在这些平台上搭建自己的嵌入式系统。

刚好有一块闲置了几年的R2S卡片机和树莓派2时代的无线网卡。


# 方案
基于最小化配置改动，尽量用docker来部署的原则。

openwrt的docker-compose文件如下，网络设置采用docker的macvlan，使得openwrt看上去像网络上的一个独立主机
```
version: '2.4'
services: 
  openwrt:
    container_name: openwrt
    image: piaoyizy/openwrt-aarch64:latest
    privileged: true
    ports:
     - 80:80
    env_file:
     - .env      
    networks:
      macnet:
        ipv4_address: 192.168.1.11
    sysctls:
      - net.ipv4.ip_forward=1
      # - net.ipv4.conf.all.rp_filter=0
    restart: unless-stopped          
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "2"

# ip link set end0 promisc on          
networks:
  macnet:
    name: macnet
    ipam:
      driver: default
      config:
        - subnet: '192.168.1.0/24'
          gateway: 192.168.1.1
    driver: macvlan
    driver_opts:
      parent: end0   
      macvlan_mode: bridge
```      

但是这样做有一个问题，是r2s反而不能访问这个docker容器（因为内核安全问题），解决的方法是增加一个本地的桥接网口，并设置路由，本机通过这个桥接网口访问，设置如下：
```
ip link set end0 promisc on
ip link add macvlan-br link end0 type macvlan mode bridge
ip addr add 192.168.1.223/32 dev macvlan-br
ip link set macvlan-br up
ip route add 192.168.1.11/32 dev macvlan-br
```

设置r2s可路由转发：
```
sudo echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf 
sudo sysctl -p

# make client from wifi can access the net 
iptables -t nat -I POSTROUTING -j MASQUERADE
```

hostapd容器的docker-compose
```
version: '2.4'
services: 
  hostapd:
    container_name: hostapd
    build: .
    image: hostapd
    cap_add: 
      - NET_ADMIN
    stop_grace_period: 3s
    network_mode: host
    env_file:
     - .env      
    volumes: 
      - ./conf/r2s/openwrt/hostapd.conf:/etc/hostapd/hostapd.conf
      - ./conf/r2s/openwrt/dhcpd.conf:/etc/dhcp/dhcpd.conf
      - ./conf/r2s/openwrt/entrypoint.sh:/entrypoint.sh
    entrypoint: ["/entrypoint.sh"] 
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "2"
```


### 以下为过时的做法，不够简练
======================================================================

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
