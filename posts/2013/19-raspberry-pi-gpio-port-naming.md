---
date: 2013-03-22
layout: post
title: Raspberry Pi GPIO 命名规则
description: Raspberry Pi GPIO naming
categories:
- Blog
tags:
- Raspberry Pi

---

树莓派和普通电脑不一样的地方在于它还带了[GPIO](http://en.wikipedia.org/wiki/General_Purpose_Input/Output)（General Purpose Input/Output），可以用来驱动各种外设（如传感器，步进电机等）。但GPIO的命名有些混乱，不同的SDK（如wiringPi，RPi.GPIO等）对GPIO的端口号取值也有两种规范（Raspberry Pi和Braodcom），下面则用图表标明了对应的叫法。

# 命名方法
1. Pin names：引脚号Pin 1 - 26等
2. Raspberry Pi names: GPIO 0, 1, 2, 3, 4 , 5 , 6 , 7等
3. Broadcom names: GPIO 17, 18, 21/27,22,23,24,25等

<img src="http://elinux.org/images/2/2a/GPIOs.png"/>
<img src="http://trouch.com/wp-content/uploads/2012/08/webiopi-chrome.png" width="600"/>

## 左排针脚

**Pin**    | **Raspberry Pi** | **Broadcom names**
------------ | ------------- | ------------
1            |    3.3V       | 3.3V
3            |    SDA0       |  I2C0 SDA
5            |    SCL0       |  I2C0 SCL
7            |    GPIO 7     |  GPIO 4
9            |    DNC        |  DNC
11           |    GPIO 0     |  GPIO 17
13           |   GPIO 2      |  GPIO 21 (rev2) / GPIO 27 (rev1)
15           |   GPIO 3      |  GPIO 22
17           |   DNC         |  DNC 
19           |    SPI MOSI   | SPI MOSI
21           |   SPI MOSO    | SPI MOSO 
23           |   SPI SCLK    | SPI SCLK
25           |   DNC         | DNC

## 右排针脚

**Pin**    | **Raspberry Pi ** | **Broadcom names**
------------ | ------------- | ------------
2           |   5V        | 5V
4           |   DNC        | DNC
6           |   GND        | GND
8           |   TX        | UART TXD
10           |  RX         | UART RXD
12           |  GPIO 1         | GPIO 18
14           |  DNC         | DNC
16           |  GPIO 4         | GPIO 23
18           |  GPIO 5         | GPIO 24
20           |  DNC         | DNC
22           |  GPIO 6         | GPIO 25
24           |  SP10 CEO N         | SP10 CEO N
26           |  SP10 CE1 N         | SP10 CE1 N  

## 根据Raspberry Pi的GPIO 查表

**Raspberry Pi GPIO**  | **Pin**  | **Broadcom GPIO** 
------------ | ------------- |  ------------- 
0           |  11 | 17        
1           |  12  | 18        
2           |  13 | 21        
3           |  15 | 22       
4           |  16 | 23        
5           |  18 | 24
6           |  22 | 26
7           |  7  | 4

Notes:

- all the UART, SPI and I2C pins can be reconfigured as GPIO if needed.


# 参考
1. http://elinux.org/RPi_Low-level_peripherals