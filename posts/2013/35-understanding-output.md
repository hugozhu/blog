---
date: 2013-05-14
layout: post
title: 树莓派的GPIO接口输出电流限制
description: Understanding output
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

树莓派提供了一个连接头让我们访问CPU的17个GPIO接口，如下图

<img src="https://www.evernote.com/shard/s26/sh/92d52938-5bd9-46a7-9b05-478e9f30f5d7/b29dcc510983784a07472c8282330b30/deep/0/Screenshot%205/14/13%208:51%20PM.png"/>

这些接口可配置成输入或输出。本文主要讨论GPIO引脚作为输出时电流的限制。

# 阻抗 (impendance)
阻抗和和电阻的区别（resistance）在于电阻的阻值是固定的，不会随着电流变化，阻抗则不然，可能随着外部变化，如电流或频率变化。从另一个角度来说，电阻是线性的，但阻抗不是。比如放大器的阻抗会随着输出的信号频率变化。

树莓派的的每个GPIO引脚都有一个寄存器可以设置引脚的驱动强度，也就是在保持输出电压为逻辑0和1的情况下，可以改变阻抗的大小从而改变GPIO引脚的输出电流大小。

通过如下电路测量相同电流下不同阻抗对应的GPIO电压输出（其中用到了一个电位器调节电流保持恒定）：

<img src="https://www.evernote.com/shard/s26/sh/11008332-acac-4625-9b6a-963c97ec7498/1e6d153774b7e95214fe0c2bba9121d8/deep/0/Screenshot%205/14/13%2010:22%20PM.png"/>

通过计算后，下表是当输出电流为2，4 … 16mA时，对应的阻抗大小以及如果发生短路时的短路电流大小。

<img src="https://www.evernote.com/shard/s26/sh/7a411df0-56b1-4f76-bce5-54961bcfcfc7/4e46d9e28d7d01a34eb9e66ee28a5ba8/deep/0/Screenshot%205/14/13%2010:39%20PM.png"/>

可以看出短路电流都是超过16mA的。

一个发光二极管压降约为1.5~2.0v，工作电流为10~20v

GPIO引脚的电流是通过板上的3.3V电压调整器输出的，树莓派是按平均每个引脚3mA来设计的，所以总的电流不能超过17 * 3 = 51mA。

# 结论

树莓派引脚电流大小的限制是：**每个引脚最大输出电流为16毫安(mA)，且同一时刻所有引脚的总输出电流不超过51毫安**

# 参考链接
1. http://www.thebox.myzen.co.uk/Raspberry/Understanding_Outputs.html