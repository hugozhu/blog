---
date: 2013-06-24
layout: post
title: 在Android上使用tcpdump
description: tcpdump on android
categories:
- Blog
tags:
- Android

---

tcpdump工具是分析网络协议和数据包的利器，也可以在Android上使用（需要root）。

首先在android上安装tcpdump

```
wget http://www.strazzere.com/android/tcpdump
adb push tcpdump /data/local/tmp/tcpdump
adb chmod 755 /data/local/tmp/tcpdump
```

然后使用root用户启动tcpdump，在android上进行相应的操作后，按ctrl+c中断

```
adb shell
shell@android:/ $ su
root@android:/ # /data/local/tmp/tcpdump -h                                    
tcpdump version 3.9.8
libpcap version 0.9.8
Usage: tcpdump [-aAdDeflLnNOpqRStuUvxX] [-c count] [ -C file_size ]
		[ -E algo:secret ] [ -F file ] [ -i interface ] [ -M secret ]
		[ -r file ] [ -s snaplen ] [ -T type ] [ -w file ]
		[ -W filecount ] [ -y datalinktype ] [ -Z user ]
		[ expression ]
root@android:/ # /data/local/tmp/tcpdump -p -vv -s 0 w /sdcard/capture.pcap
```
tcpdump会在/sdcard下生成文件，可以通过`adb pull /sdcard/capture.pcap`把文件传到PC上用[wireshark](http://wireshark.org/)看，也可以直接在android上通过[SharkReader](https://play.google.com/store/apps/details?id=lv.n3o.sharkreader)看。


# 参考链接
1. http://www.kandroid.org/online-pdk/guide/tcpdump.html
2. http://wireshark.org/
3. http://www.strazzere.com/blog/2009/08/gather-packets-from-your-android-without-arp-spoofing/