---
date: 2013-04-18
layout: post
title: 树莓派I2C编程
description: I2C Programming on Raspberry Pi
categories:
- Blog
tags:
- Raspberry Pi

---

除了[SPI](http://hugozhu.myalert.info/2013/04/05/25-get-spi-working-on-raspberry-pi-spi.html)协议外，树莓派还支持[I2C](http://zh.wikipedia.org/wiki/I²C)。I2C是为了连接低速周边装置设计的，只需要用两根线（SDA和SCL，也就是树莓派的端口8和9-wiringPi编号）。

# I2C

<img src="http://upload.wikimedia.org/wikipedia/commons/thumb/3/3e/I2C.svg/350px-I2C.svg.png"/>

上图是一个主控使用I2C驱动3个设备的示意图


# 参考链接

1. http://zh.wikipedia.org/wiki/I²C
2. https://projects.drogon.net/raspberry-pi/wiringpi/i2c-library/