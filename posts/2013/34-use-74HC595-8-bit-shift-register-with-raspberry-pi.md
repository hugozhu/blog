---
date: 2013-04-14
layout: post
title: 使用8位移位寄存器74HC595扩展树莓派的IO端口
description: Use 74HC595 8bit shift register with Rapsberry Pi
categories:
- Blog
tags:
- Raspberry Pi

---

树莓派的GPIO接口数目有限，驱动一个步进电机需要占用4个， 一个Nokia 5110液晶也要占4个， 传感器输入至少需要一个，多玩几个外设后接口就不够用了。如果接口可以复用就可以让树莓派驱动更多的外设了，本文讨论如何使用74HC595集成电路芯片来扩展树莓派的I/O接口。


# 芯片介绍
SN74HC595N是德州仪器公司生产的集成电路芯片，是一个8位串行输入变串行输出或并行输出移位寄存器，具有高阻关断，高电平和低电平三态输出。在IO扩充上，可以最多串联15片，也就是高达120个IO扩充。

<img src="https://www.evernote.com/shard/s26/sh/b90034cc-cdf8-426d-a79d-16bbbcbb49b2/0ba3371f3039afa71a17bad845eede7a/deep/0/Screenshot%205/12/13%2010:42%20PM.png"/>

（注意到芯片上的小凹槽了吗，拿芯片的时候以这个为参考物就不会搞反了）

接口的常用命名方式有以下两种：

接口代号(编号)  | 说明     | 接口代号(编号)      | 说明 
---------- | -------- | ------------  | ------------ 
Q7'(9)   |serial data output    | QH'  (9)           | serial data output 
MR (10)	 |Master Reset (Active Low)    | SRCLR  (10)            | Shift register CLeaR
SH_CP (11)          |shift register clock input    | SRCLK  (11)            | Shift Register CLocK input
ST_CP (12)          |storage register clock input    | RCLK  (12)            | storage Register CLocK input
OE (13)          |output enable input (Active Low)| OE  (13)            | Output Enable
DS (14)         |serial data input       | SER (14)             | SERial data input
Qx (15，1-7)         |data output       | Qx (15，1-7)            | data output

# 控制流程
如果要在8个引脚输出01010101

1. 将Pin 14（DS, SER）置为高电平（1）；
2. 将Pin 11 (SH_CP, SRCLK))做高低电平切换，形成一个脉冲信号，这个信号会将数据从移位寄存器C1移动到下一个移位寄存器C2，。。。
3. 接着将Pin 14设为低电平（0），再将Pin 11做1->0的脉冲变化，将数据继续往下移，依次类推直到8位都输入完成；
4. 将Pin 12（ST_CP, RCLK）做1->0的脉冲，将8位数据一次并行输出。

如果要串联多片，由上一片的Pin 9接到下一片的Pin 14即可，这样输入16 bit后，再向Pin 12输入一个1->0的脉冲，16 bit会并行输出。

如果要一次清除所有数据，将Pin 10设为低电平后，再向Pin 12输入一个1->0的脉冲即可；

Pin 13还有高阻关断的第三态输出功能。

# 材料
1. [74HC595](http://item.taobao.com/item.htm?spm=a1z0d.1.1000638.45.bTo9PZ&id=3136071980) 因为价格便宜，建议一次买3~10片
2. 面包板
3. 面包板跳线 14根
4. LCD 8个
5. 300欧姆电阻 8个

# 参考链接
1. http://baike.baidu.com/view/1309513.htm
2. http://ruten-proteus.blogspot.com/2012/11/io-74hc595-ic.html
3. http://en.wikipedia.org/wiki/Charlieplexing