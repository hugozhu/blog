---
date: 2013-03-06
layout: post
title: Raspberry Pi做BT下载机+高清播放器
description: Raspberry Pi with XMBC
categories:
- Blog
tags:
- Raspberry Pi

---

## 介绍

首先高清播放器功能只是Raspberry Pi的一个小功能，如果你只需要高清播放功能又不想折腾，那还是买个[山寨的](http://s.taobao.com/search?q=%B8%DF%C7%E5%B2%A5%B7%C5%C6%F7&commend=all&ssid=s5-e&search_type=item&sourceId=tb.index&initiative_id=tbindexz_20130306)的更简单。。。

Raspberry Pi的图形处理器规格：Broadcom VideoCore IV, OpenGL ES 2.0, 1080p 30 h.264/MPEG-4 AVC 高清解码器，内存和CPU共享（可设置成256M），性能还是很强劲的。HDMI支持640x350和1920×1200（1080P）的分辨率。安装了[XBMC](http://xbmc.org)，基本上可以实现包括Airplay在内的Apple TV上的大部分功能，但价格只有其一半不到，可以播放下载的电源或观看在线视频，如[一搜](http://yisou.com)，优酷，搜狐视频，奇艺等。

## 外设
除了Pi单片机外，你还需要以下外设附件：

1. 5V-1A左右的电源，可以用iPhone或iPad的充电电源，或手机的，电流最少要800毫安
2. HDMI线一根，接电视机
3. SD一张，最少2G

## Raspbmc

[Raspbmc](http://www.raspbmc.com/) 是专为在Raspberry Pi上运行[XBMC](http://xbmc.org)的定制Linux。最小化的安装，减少了不必要的软件和资源占用，简化了安装和配置，没有Linux知识也可以上手。这个版本的维护者是一个19岁的小朋友Sam Nazarko。有时间折腾的同学可以自己编译XMBC安装。

### 特点：
1. 免费，开源
2. 支持多语言
3. 支持1080P回放
4. 支持直接播放NFS，SMB，FTP,HTTP或USB硬盘的有视频文件，支持大多数格式
5. 支持AirPlay或AirTune功能，可以把iPhone/iPad上的视频或音乐通过Pi投放到电视上，这点和Apple TV功能一样
6. 支持GPIO
7. 基于Debian，可以从Debian的软件源安装其它软件
8. 支持1080P DTS软解，这个不少播放器是不支持的，需要额外License
9. 内置了以下服务:
    1. Samba
    2. TVHeadend Server
    3. FTP Server
    4. SSH Server

### 安装
1. Windows下载[安装程序](http://download.raspbmc.com/downloads/bin/installers/raspbmc-win32.zip)，运行即可。

    ![image](http://www.raspbmc.com/wp-content/uploads/2012/06/ins-300x165.jpg)
2. Linux/Mac:

    ```
    curl -O http://svn.stmlabs.com/svn/raspbmc/testing/installers/python/install.py
    chmod +x install.py
    sudo python install.py
    
    ```
   ![image](http://www.raspbmc.com/wp-content/uploads/2012/06/installPython.png)
3.  或直接下载[安装包](http://download.raspbmc.com/downloads/bin/ramdistribution/installer.img.gz)安装

### 下载
你可以在Pi上外接一个USB移动硬盘，但要注意硬盘要有自己电源，也可以mount网络上的硬盘分区。然后运行transmission软件下载视频。

#### Transmission
1. 安装

    ```
    sudo apt-get install transmission-daemon
    sudo /etc/init.d/transmission-daemon stop
    sudo nano /etc/transmission-daemon/settings.json
    ```
2. 配置

   ```
    “rpc-whitelist”: “127.0.0.1″, to “rpc-whitelist”: “*.*.*.*”,
    “rpc-password”: “password”, to “rpc-password”: “替换成管理密码“,
    “rpc-username”: “username”, to “rpc-username”: “替换成管理用户“,   
    “download-dir”：“\/home\/xbmc\/Videos\/Downloads”,
   ```

   ```
    sudo chmod g+rw /home/xbmc/Videos/Downloads
    sudo chgrp -R debian-transmission /home/xbmc/Videos/Downloads   
    sudo /etc/init.d/transmission-daemon start
   ```   
3. 24x7开始下载，耗电量很低的，这是下载界面：
    ![image](https://pbs.twimg.com/media/BErnJ-6CcAEVYsV.jpg:large)

### 遥控和播放

遥控方案有两种：

1. 红外接收器 + 电视/DVD/VCD/EVD等已有遥控器，[这里](http://forum.stmlabs.com/showthread.php?tid=5549)有一个实现方案
       
2. 通过网络用手机来遥控，其实就是用任何一个xmbc的客户端
   1. iPhone: [Offical XBMC Remote](https://itunes.apple.com/us/app/unofficial-official-xbmc-remote/id520480364?ls=1&mt=8) 免费的
   2. Android: [Android XBMC Remote](http://code.google.com/p/android-xbmcremote/) 免费的

### 参考文章:

1. http://www.raspbmc.com/about/
