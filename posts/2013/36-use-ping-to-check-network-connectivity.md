---
date: 2013-06-01
layout: post
title: 使用Ping来检查网络连通性
description: Use ping to check network connectivity
categories:
- Blog
tags:
- Shell Scripting

---

树莓派使用了一个无线网卡连接家里的无线路由器，在实际使用过程中发现连续运行多天后会掉线，而且掉线后基本上就再也连不上网了，需要重启树莓派才能恢复，十分麻烦。

假设无线路由器IP是192.168.1.1，于是每隔15分钟检查一下，是否能从树莓派上ping通路由器；如果不能则重启无线网络，脚本如下：


**network.sh**

```
#!/bin/bash

export PATH=/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin

ping_count() {
  count=0
  `timeout 5 ping 192.168.1.3 | while read LINE; do
  {
        if [[ "${LINE}" =~ "64 bytes from" ]]; then
                let "count = $count + 1"
                echo "export count=$count"
        fi
  }
  done`
  echo $count
}


if [[ $(ping_count) < 1 ]]; then
        ifconfig wlan0
        ifconfig wlan0 down
        sleep 1
        ifconfig wlan0 up
        sleep 1
        netcfg -r wlan0-Hugo-Nas
        sleep 5
        if [[ $(ping_count) < 1 ]]; then
                echo "Fatal error: wifi is down, rebooting now..."
                reboot
        fi
fi
```

可以用ifconfig wlan0 down && ./network.sh来测试脚本是否能正常工作，测试完成后就可以放到crontab里执行了。

```
*/60 * * * * /home/hugo/bin/network.sh >> /mnt/usb/logs/network.log 2>&1
```

这样出差不在家也不用通知家里的人拔插树莓派电源了。。。