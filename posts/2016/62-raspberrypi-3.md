---
date: 2016-04-13
layout: post
title: 树莓派3初体验
description: Raspberry Pi 3
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

树莓派基金会今年推出的新品Raspberry Pi 3已经到手，官方宣传性能比树莓派2代快50%，比树莓派1代快10倍，第一次使用的64位四核处理器（博通BCM2837）配备了ARM Cortex-A53处理器，内置了802.11 b/g/n 2.4GHz WIFI和蓝牙4.1，显卡没变，还是双核VideoCore IV（并不支持4K视频）。CPU升级也对能耗有了更高的要求，官方说法最好是使用5V 2.5A的电源输入（iPad充电器），实测在无外设的情况下，2A的电流输出也可以让Pi 3正常运行。价格和树莓派2也一样，官方售价税前35美金，国内可以在淘宝上买到，238元一枚，[点此购买](https://item.taobao.com/item.htm?spm=a230r.1.14.19.82ysfy&id=527525039334&ns=1&abbucket=18#detail)

<img src="http://files.linuxgizmos.com/rpi_pi3_detail.jpg" width=500/>


**我深深的认为每个程序员都需要有一块树莓派，24*7的运行在家里的网络上** ，投入成本在300人民币以内（树莓派加电源：250，SD卡：50），每年电费在10元以内。树莓派支持各种编程语言的开发，安装体验各种操作系统非常简单，丰富的外部接口，支持很多类型的传感器和控制器外设，可以让你轻松设计和实现智能硬件，技术让生活更美好~

# 外观

树莓派3的规格大小则和树莓派2完全一样，你甚至可以直接用树莓派2的外壳，完全贴合。

![image](http://ww4.sinaimg.cn/bmiddle/6bc40342gw1f1sd7r0iwrj20zk0qodnp.jpg)


背面看略有不同，中间的芯片是1G内存，树莓派3的内存速度快了1倍，右边是CF卡插槽，树莓派2是回弹式卡槽（取出CF卡时只要往力再摁一下就会弹出），树莓派3可能为了降低成本或是因为板卡空间的问题改成了更紧凑的插入式，装上外壳后取出的时候有点费劲，我需要用瑞士军刀的镊子夹出来。

![image](http://ww2.sinaimg.cn/bmiddle/6bc40342gw1f1sd7pk73sj20qo0zkdpq.jpg)

（下图是树莓派3）

# 安装启动

如果你不需要GUI,推荐安装[Raspbian Jessie Lite](https://downloads.raspberrypi.org/raspbian_lite_latest)，大小只有298.3M，下载完成后解压缩成img文件后用dd命令写到CF卡上去。你也可以下载[NOOBS](https://downloads.raspberrypi.org/NOOBS_latest)完整安装。


# WIFI


编辑无线配置文件`/etc/wpa_supplicant/wpa_supplicant.conf`

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
	ssid="<无线AP的名称>"
   psk="<无线密码>"
}
```

如果有多个无线网络可以接入，增加`network`即可

修改好后如果没有自动生效连上网络可以通过以下命令强制生效

```
ifdown wlan0
ifup wlan0
```

非常重要的一点是：如果你希望无线网络一直连接，需要关闭掉无线模块的电源管理，否则没有网络流量后，WIFI会自动关闭。。。

```
sudo iwconfig wlan0 power off
```

测试wifi速度：从通过网线连接到极路由2的Mac拷贝文件到树莓派3上，内置网卡的传输速度在4MB/s左右，如果换上300Mbps的USB无线网卡`EDUP EP-N1557`则可以达到9MB/s左右，速度要快1倍。结论是内置的wifi模块速度只有150Mbps，如果对网络速度要求较高，最稳定的方法还是插根网线。

安装好`nginx`后，用`ab`测试最简单的HTML网页性能，树莓派3可以轻松超过1000 QPS，满足个人网站的性能需求。

# CPU
树莓派3使用的4核Cortex-A53 BCM2837 SoC为了向下兼容，架构上和树莓派2使用的4核Cortex-A7 BCM2836差不多，在32位模式运行下，速度要快50~60%（时钟频率1.2GHz vs 900MHz），尽管CPU已经支持64位，官方的操作系统Raspbian还是32位，当然树莓派3内存只有1G，运行64位操作系统可能会有点累,。
如果要体验真64位系统，目前可以考虑带2G内存，千兆网口的[ODROID-C2](https://item.taobao.com/item.htm?spm=a230r.1.14.1.aQSAa0&id=527695599811&ns=1&abbucket=18#detail)，可装Ubuntu 16.04和Android 5.1。

一般情况下，不带散热片的CPU温度在45度左右，我有个脚本会每五分钟上传一下温度：[https://personal.xively.com/feeds/1480103458](https://personal.xively.com/feeds/1480103458)


# Node.js

安装5.0.0

```
wget https://nodejs.org/dist/v5.0.0/node-v5.0.0-linux-armv7l.tar.gz
tar -xvf node-v5.0.0-linux-armv7l.tar.gz 
cd node-v5.0.0-linux-armv7l
sudo cp -R * /usr/local/

```

升级

```
sudo npm install npm -g
sudo npm install -g n
sudo n latest
```

# Golang

```
wget https://storage.googleapis.com/golang/go1.6.linux-armv6l.tar.gz
tar zxvf go1.6.linux-armv6l.tar.gz
sudo mv go /usr/local
```

后面主要研究下如何使用树莓派3的蓝牙功能，能否和Apple Watch配对呢。。。

