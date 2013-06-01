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

```
#!/bin/bash

export PATH=/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin
count=0
`timeout 5 ping 192.168.1.1 | while read LINE; do
{
        if [[ "${LINE}" =~ "64 bytes from" ]]; then
                let "count = $count + 1"
                echo "export count=$count"
        fi
}
done`

if [[ $count < 1 ]]; then
        netcfg -r wlan0-Hugo-Nas
fi
```

这样出差不在家也不用通知家里的人插拔电源了。。。