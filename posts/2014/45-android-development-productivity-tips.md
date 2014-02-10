---
date: 2014-02-10
layout: post
title: 提高Android开发效率的小贴士
description: Android development productivity tips
categories:
- Blog
tags:
- Android

---

# 查看日志 adb logcat

下面命令将只显示错误日志，和所有Tag＝mytag的调试日志，-C 会用不同颜色区分不同级别的日志，但只有Android 4.3以后才支持。

```
adb logcat [-C] *:E <mytag>:D
```

# 远程调试 adb over TCP

首先在手机或Pad上执行以下命令（要求root）

```
su
setprop service.adb.tcp.port 5555
stop adbd
start adbd
```
再执行下面命令则可以看到手机的网络地址
```
netcfg | grep wlan
```

在电脑上则执行

```
adb connect <mobile_phone_ip> 5555
adb shell
```
