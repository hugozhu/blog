---
date: 2013-03-02
layout: post
title: OpenVPN使用多个端口
categories:
- Blog
tags:
- Linux
- VPN

---

Openvpn本身不能设置多个端口，使用iptables可以解决这个问题 （假设openvpn本来56788端口）：

    for port in {56780..56787}
    do 
        iptables -t nat -A PREROUTING -p tcp -d <your_external_ip> --dport $port -j REDIRECT --to-port 56788
    done
