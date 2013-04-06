---
date: 2013-03-22
layout: post
title: Raspberry Pi GPIO的编号规范
description: Raspberry Pi GPIO numbering
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

树莓派和普通电脑不一样的地方在于它还带了17个可编程的[GPIO](http://en.wikipedia.org/wiki/General_Purpose_Input/Output)（General Purpose Input/Output），可以用来驱动各种外设（如传感器，步进电机等）。但GPIO的编号方法有些混乱，不同的API（如wiringPi，RPi.GPIO等）对GPIO的端口号编号并不一样，下面则用图表标明了对应的叫法，这样在看程序例子的时候可以确定物理是哪个接口。

# GPIO库
1. [wiringPi](https://github.com/WiringPi/WiringPi) C，有Perl, PHP, Ruby, Node.JS和**[Golang](http://github.com/hugozhu/rpi)**的扩展，支持wiringPi Pin和BCM GPIO两种编号
2. [RPi.GPIO](https://pypi.python.org/pypi/RPi.GPIO) Python，支持Board Pin和BCM GPIO两种编号
3. [Webiopi](http://code.google.com/p/webiopi/)，Python, 使用BCM GPIO编号
4. [WiringPi-Go](http://github.com/hugozhu/rpi), Go语言，支持以上三种编号

# 编号规范
1. 第一列是wiringPi API中的缺省编号，`wiringPiSetup()`采用这列编号
2. 第二列（Name）往往是转接板的编号
3. 第三列是树莓派板子上的自然编号（左边引脚为1-15，右边引脚为2-26），`RPi.GPIO.setmode(GPIO.BOARD)`采用这列编号
4. 树莓派主芯片提供商Broadcom的编号方法，相当于调用了`WiringPiSetupGpio()`或`RPi.GPIO.setmode(GPIO.BCM)`采用这列编号

wiringPi Pin  | Name     | Board Pin     | BCM GPIO
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
13         |MISO      | 21            | 9
14         |SCLK      | 23            | 11
15         |TXD       | 8             | 14
16         |RXD       | 10            | 15


Rev.2 新增的引脚：

wiringPi Pin | Name     | Board Pin     | BCM GPIO
---------- | -------- | ------------  | ------------ 
17         |GPIO 8    |             | 28   
18         |GPIO 9    |             | 29   
19         |GPIO10    |             | 30   
20         |GPIO11    |             | 31   


<img src="https://pbs.twimg.com/media/BGBhJ4LCAAA50eS.jpg:large"  width="600"/>

## GPIO转接板
GPIO转接板通过彩虹排线可将树莓派的GPIO引脚转接到面包板上，方便试验，下图是一个相应的产品，可以看到每个引脚标都已标注好了名称，查上表就知道代码里该用哪个编号做参数了。

<img src="http://img03.taobaocdn.com/imgextra/i3/21288305/T23BjrXfJaXXXXXXXX_!!21288305.jpg"/>

## 物理左排针脚说明

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

## 物理左排针脚说明

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