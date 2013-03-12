---
date: 2013-03-13
layout: post
title: 用发光二极管提示网站首页有访客到来
description: 客人来了发光二极管会闪0.2秒
categories:
- Blog
tags:
- Raspberry Pi

---

本方法只适合小网站，主要是好玩。

# 硬件安装
<img src="http://ww3.sinaimg.cn/bmiddle/6bc40342jw1e2ni9esb2uj.jpg"/>


# 软件准备

``` 
pacman -S python2
pacman -S python2-distribute
    
```

# [Webiopi](http://code.google.com/p/webiopi/)
这是一个使用RESTful API控制Pi的GPIO，使用起来非常简单

<img src="http://trouch.com/wp-content/uploads/2012/08/webiopi-chrome.png"/>


# 代码脚本

```
hugo@raspberrypi ~/bin $ cat traffic_led.sh 
#!/bin/sh

tail -f  /mnt/usb/logs/nginx/access.log  | grep --line-buffered "GET / HTTP" | while read LINE; do  {
   #echo $LINE
   curl -s --data "" "http://webiopi:raspberry@raspberrypi2:8000/GPIO/23/value/0"
   sleep 0.2
   curl -s --data "" "http://webiopi:raspberry@raspberrypi2:8000/GPIO/23/value/1"
   sleep 0.2
}
done
```