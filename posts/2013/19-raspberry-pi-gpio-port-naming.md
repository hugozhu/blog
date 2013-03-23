---
date: 2013-03-22
layout: post
title: Raspberry Pi GPIO的编号规则
description: Raspberry Pi GPIO naming
categories:
- Blog
tags:
- Raspberry Pi

---

树莓派和普通电脑不一样的地方在于它还带了[GPIO](http://en.wikipedia.org/wiki/General_Purpose_Input/Output)（General Purpose Input/Output），可以用来驱动各种外设（如传感器，步进电机等）。但GPIO的编号方法有些混乱，不同的API（如wiringPi，RPi.GPIO等）对GPIO的端口号编号并不一样，下面则用图表标明了对应的叫法。


# wiringPi和RPi.GPIO中编号
1. 第一列是wiringPi API中的编号
2. 第二列（Name）往往是转接板的编号
3. 树莓派板子上的编号是第三列，相当于调用了RPi.GPIO.setmode(GPIO.BOARD)
4. 树莓派主芯片提供商Broadcom的编号方法，相当于调用了RPi.GPIO.setmode(GPIO.BCM)

wiringPi   | Name     | GPIO.BOARD    | GPIO.BCM
---------- | -------- | ------------  | ------------ 
0          |GPIO 0    | 11            | 17 
1          |GPIO 1    | 12            | 18
2          |GPIO 2    | 13            | 21
3          |GPIO 3    | 15            | 22
4          |GPIO 4    | 16            | 23
5          |GPIO 5    | 18            | 24
6          |GPIO 6    | 22            | 25
7          |GPIO 7    | 7             | 4
8          |SDA       | 3             | 0
9          |SCL       | 5             | 1
10         |CE0       | 24            | 8
11         |CE1       | 26            | 7
12         |MOSI      | 19            | 10
13         |MOSO      | 21            | 9
14         |SCLK      | 23            | 11
15         |TXD       | 8             | 14
16         |RXD       | 10            | 15


# 编号方法
1. Pin names：引脚号Pin 1 - 26等
2. Raspberry Pi names: GPIO 0, 1, 2, 3, 4 , 5 , 6 , 7等
3. Broadcom names: GPIO 17, 18, 21(树莓派第二版)或27(树莓派第一版), 22, 23, 24, 25等

<img src="https://pbs.twimg.com/media/BGBhJ4LCAAA50eS.jpg:large"/>

## 使用GPIO转接板
<img src="http://img03.taobaocdn.com/imgextra/i3/21288305/T23BjrXfJaXXXXXXXX_!!21288305.jpg"/>
如果使用GPIO转接板, 板上的编号对应为下表


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

**Pin**    | **Raspberry Pi** | **Broadcom names**
------------ | ------------- | ------------
2           |   5V        | 5V
4           |   DNC        | DNC
6           |   GND        | GND
8           |   TX        | UART TxD
10           |  RX         | UART RxD
12           |  GPIO 1         | GPIO 18
14           |  DNC         | DNC
16           |  GPIO 4         | GPIO 23
18           |  GPIO 5         | GPIO 24
20           |  DNC         | DNC
22           |  GPIO 6         | GPIO 25
24           |  SP10 CEO N         | SP10 CEO N
26           |  SP10 CE1 N         | SP10 CE1 N  


Notes:

- all the UART, SPI and I2C pins can be reconfigured as GPIO if needed.


# 参考
1. http://elinux.org/RPi_Low-level_peripherals
2. https://projects.drogon.net/raspberry-pi/wiringpi/pins/