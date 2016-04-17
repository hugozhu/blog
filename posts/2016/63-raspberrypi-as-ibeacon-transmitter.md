---
date: 2016-04-19
layout: post
title: 树莓派3变身iBeacon发射器
description: Raspberry Pi 3 as iBeacon Transmitter
categories:
- Blog
tags:
- Raspberry Pi
- Bluetooth

---

{:toc}

iBeacon是apple公司提出的“一种可以让附近手持电子设备检测到的一种新的低功耗、低成本信号传送器”的一套可用于室内定位系统的协议。iBeacon技术通过低功耗蓝牙（BLE），也就是我们所说的智能蓝牙来实现。


# 设置
树莓派3内置了蓝牙芯片，最新的Raspian也已经安装好`bluez`，所以基本上不用什么设置，就可以把树莓派3当做iBeacon基站来使用了。

通过以下命令可以确认蓝牙芯片工作正常

```
hugo@raspberrypi3:~ $ sudo hciconfig
hci0:	Type: BR/EDR  Bus: UART
	BD Address: B8:27:EB:BF:E0:C5  ACL MTU: 1021:8  SCO MTU: 64:1
	UP RUNNING 
	RX bytes:3841 acl:0 sco:0 events:239 errors:0
	TX bytes:5213 acl:0 sco:0 commands:237 errors:0
```	

# 脚本
iBeacon使用的是BLE技术，具体而言，利用的是BLE中名为“通告帧”（Advertising）的广播帧。通告帧是定期发送的帧，只要是支持BLE的设备就可以接收到。iBeacon通过在这种通告帧的有效负载部分嵌入苹果自主格式的数据来实现。
iBeacon的数据主要由四种资讯构成，分别是UUID（通用唯一标识符）、Major、Minor、Measured Power。
UUID是规定为ISO/IEC11578:1996标准的128位标识符。
Major和Minor由iBeacon发布者自行设定，都是16位的标识符。比如，连锁店可以在Major中写入区域资讯，可在Minor中写入个别店铺的ID等。另外，在家电中嵌入iBeacon功能时，可以用Major表示产品型号，用Minor表示错误代码，用来向外部通知故障。
Measured Power是iBeacon模块与接收器之间相距1m时的参考接收信号强（RSSI：Received Signal Strength Indicator）。接收器根据该参考RSSI与接收信号的强度来推算发送模块与接收器的距离。


iBeacon发射的信号格式如下：

## 消息头

```
1E
02      # Number of bytes that follow in first AD structure
01      # Flags AD type
1A      # Flags value 0x1A = 000011010  
          bit 0 (OFF) LE Limited Discoverable Mode
          bit 1 (ON) LE General Discoverable Mode
          bit 2 (OFF) BR/EDR Not Supported
          bit 3 (ON) Simultaneous LE and BR/EDR to Same Device Capable (controller)
          bit 4 (ON) Simultaneous LE and BR/EDR to Same Device Capable (Host)
1A      # Number of bytes that follow in second (and last) AD structure
```

##  Vendor的标识

```
FF      # Manufacturer specific data AD type
4C 00   # Company identifier code (0x004C == Apple)
02      # Byte 0 of iBeacon advertisement indicator
15      # Byte 1 of iBeacon advertisement indicator
```


##  通告帧信息

```
F6 BC 15 E0 93 90 46 67 9B E1 86 6E C8 A1 99 DC # our iBeacon proximity uuid
00 00   # Major 
00 00   # Minor 
C8 00   # Calibrated Tx power
```

以下脚本会随机生成一个UUID和相应的iBeacon发射命令

```
import sys;
import uuid;
s=uuid.uuid4().hex
#s="f6bc15e0939046679be1866ec8a199dc"
sys.stdout.write("UUID:\t\t"+s+"\n")
sys.stdout.write("Proximity UUID:\t"+s[0:8]+'-'+s[8:12]+'-'+s[12:16]+'-'+s[16:20]+'-'+s[20:]+"\n")
sys.stdout.write("Cmd:\t\thcitool -i hci0 cmd 0x08 0x0008 1E 02 01 1A 1A FF 4C 00 02 15 "+
	s[0:2]  +' '+s[2:4]  +' '+s[4:6]  +' '+s[6:8]  +' '+
	s[8:10] +' '+s[10:12]+' '+s[12:14]+' '+s[14:16]+' '+
	s[16:18]+' '+s[18:20]+' '+s[20:22]+' '+s[22:24]+' '+
	s[24:26]+' '+s[26:28]+' '+s[28:30]+' '+s[30:32]+' '+
	"00 00 00 00 C8 00\n")
```	

## 发射信号命令

```
sudo hciconfig hci0 up
sudo hciconfig hci0 leadv 3
sudo hciconfig hci0 noscan
sudo hcitool -i hci0 cmd 0x08 0x0008 1E 02 01 1A 1A FF 4C 00 02 15 f6 bc 15 e0 93 90 46 67 9b e1 86 6e c8 a1 99 dc 00 00 00 00 C8 00
```

## 发现iBeacon信号
用iPhone安装一个免费app `Beacon Toolkits` [http://localz.com/blog/beacon-toolkit/](http://localz.com/blog/beacon-toolkit/)

如下增加一个iBeacon UUID：

<img src="http://ww1.sinaimg.cn/mw690/6bc40342gw1f22i748rigj20ku112tah.jpg"/>

选择扫描后就会出现下图了，可以看到距离很近哦~

<img src="http://ww4.sinaimg.cn/mw690/6bc40342gw1f22i74jof2j20ku112wfs.jpg"/>

# App使用iBeacon获取推送信息

iOS 9可以在后台感知附近的iBeacon基站并通知用户，这给了开发者无限的想象空间，而现在你就可以用树莓派3创建一个自己的iBeacon基站了。


# 参考链接

1. https://zh.wikipedia.org/wiki/IBeacon
2. http://www.wadewegner.com/2014/05/create-an-ibeacon-transmitter-with-the-raspberry-pi/
3. http://www.3pillarglobal.com/insights/ios-corelocation-framework-beacon-detection
4. http://developer.estimote.com/ibeacon/