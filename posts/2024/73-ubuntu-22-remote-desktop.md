---
date: 2024-02-20
layout: post
title: Ubuntu 22 Remote Desktop Sharing Without Real Monitor
description: Ubuntu 22 remote desktop sharing without monitor.
categories:
- Blog
tags:
- Ubuntu Linux

---

{:toc}

## xrdp solution
```
sudo apt-get install xserver-xorg-video-dummy
sudo apt-get install xserver-xorg-core

Download Microsoft Remote Desktop client
```

*/usr/share/X11/xorg.conf.d/xorg.conf*
```
Section "Device"
    Identifier "DummyDevice"
    Driver "dummy"
    VideoRam 256000
EndSection

Section "Screen"
    Identifier "DummyScreen"
    Device "DummyDevice"
    Monitor "DummyMonitor"
    DefaultDepth 24
    SubSection "Display"
        Depth 24
        Modes "1920x1080_60.0"
    EndSubSection
EndSection

Section "Monitor"
    Identifier "DummyMonitor"
    HorizSync 30-70
    VertRefresh 50-75
    ModeLine "1920x1080" 148.50 1920 2448 2492 2640 1080 1084 1089 1125 +Hsync +Vsync
EndSection
```

```
ln -s  /usr/share/X11/xorg.conf.d/xorg.conf /etc/X11
```

## 关掉账号自动登录
非常重要！

## 还原下面的文件内容来使用物理显示器
```
/usr/share/X11/xorg.conf.d/xorg.conf
/etc/X11/xorg.conf 
```