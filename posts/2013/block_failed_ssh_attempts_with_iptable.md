---
date: 2013-03-08
layout: post
title: 如何封杀尝试Raspberry Pi SSH密码的来源IP
description: Block failed ssh attempts with iptable
categories:
- Blog
tags:
- Linux
- Network

---

Raspberry Pi整天开着，如果用缺省SSH端口对外开放，就会经常遇到扫描SSH密码的肉鸡。虽然密码不是很简单，但还是感觉很不安全的。

系统的ssh登录日志文件在：/var/log/auth.log，登录失败时会记录以下格式的日志：

    Mar  7 10:31:51 raspberrypi sshd[24510]: Failed password for root from 221.8.19.129 port 4066 ssh2
    Mar  7 10:31:55 raspberrypi sshd[24514]: Failed password for root from 221.8.19.129 port 4079 ssh2
    Mar  7 10:31:56 raspberrypi sshd[24518]: Failed password for sshd from 221.8.19.129 port 4080 ssh2
    Mar  7 10:32:26 raspberrypi sshd[24522]: Failed password for sshd from 221.8.19.129 port 4149 ssh2


用最简单的Shell脚本来解决这个问题：

### guard.sh

    #!/bin/bash
    
    last_ip=""
    tail -f /var/log/auth | while read LINE; do
    {
        if [[ "${LINE}" =~ "Failed" ]]; then            
            ip="$(echo ${LINE} | awk '{print $(NF-3)}')"
            if [[ "$last_ip" == "$ip" ]]; then
                 echo "block $ip"
                 #curl -s --data-ascii "uuid=<my iphone's uuid>" --data "body=${LINE}" http://raspberrypi/pushme                 
                 iptables -A INPUT -s "$ip" -j DROP
            fi
            last_ip=$ip
            echo $LINE
        fi
    }
    done
        
用root用户执行以下命令，也可以放到启动脚本里：/etc/rc.local

    nohup /root/bin/guard.sh > /var/logs/guard.log 2>&1 &

如果连续两次输错密码，那ip就会被封，我另外加了一个报警，会通知到我的手机，这下感觉安全了些。

如果需要解封这些IP，可以用命令```iptables -F```

脚本还很简单，还可以有不少改进，可以在评论里讨论。