---
date: 2013-03-09
layout: post
title: 在Raspberry Pi上安装ArchLinux
description: Setup ArchLinux On Raspberry Pi
categories:
- Blog
tags:
- Raspberry Pi

---

## 介绍
之前买的Raspberry Pi因为要跑[这个网站](http://hugozhu.myalert.info)，不能经常拔下来玩别的，所以又买了一个，这次安装的是[Arch Linux](https://www.archlinux.org)。这个发行版安装好后非常基础，占用的空间也只有600M不到，比较合适已有Linux基础的同学玩。初学者可以玩官方推荐的[Raspbian](http://www.raspbian.org)。

Arch Linux特点：
1. 启动快，上电后只要3s完成启动
2. 安装完没有图形界面，干净
3. 面向开发者的系统
4. 包管理系统pacman很好用，一个命令就可以完成各种操作
5. ArchLinux缺省账号和密码是root/root，弄好了后要记得修改root密码
6. 从中国用下载包很快，比Raspbian的源快多了

## 增加管理员用户
```
   useradd hugo
   passwd hugo
   mkdir /home/hugo
   chown hugo:hugo /home/hugo   
   pacman -S sudo
   visudo   
   
```
执行`visudo`把新用户设置成管理员（增加sudo权限），最后面增加下面一行：

```
    hugo ALL=(ALL) NOPASSWD: ALL
```

## USB盘
插上USB盘后，ArchLinux并不会自动mount，手动mount的过程如下:
插上USB前后执行两次 `lsblk -o name,kname,uuid`，那么输出上多出的那行就是该USB的设备名，或UUID，找到该行后就可以执行mount命令了（注意sda这个符号不同机器可能不一样）

```
    [root@raspberrypi2 ~]# lsblk -o name,kname,uuid   
    NAME        KNAME     UUID
    sda         sda       001B-9622
    mmcblk0     mmcblk0   
    ├─mmcblk0p1 mmcblk0p1 44C8-CEF1
    └─mmcblk0p2 mmcblk0p2 fcee8534-f5f0-42ee-83ac-f943f878ee67
    
    mkdir /mnt/usb
    mount /dev/sda /mnt/usb 
    或 
    mount -U 001B-9622 /mnt/usb
```

格式化整个USB盘可以用`mkfs.ext4 /dev/sda`
然后在/etc/fstab里增加一行，以后重启就会自动mount了：

    /dev/sda       /mnt/usb        ext4    defaults,noatime  0       0

还可以测试一下SD卡和USB盘的读写性能：
```
[root@raspberrypi2 ~]# hdparm -Tt /dev/mmcblk0

/dev/mmcblk0:
 Timing cached reads:   292 MB in  2.00 seconds = 145.73 MB/sec
 Timing buffered disk reads:  48 MB in  3.12 seconds =  15.38 MB/sec

[root@raspberrypi2 ~]# hdparm  -Tt /dev/sda

/dev/sda:
 Timing cached reads:   280 MB in  2.00 seconds = 139.80 MB/sec
 Timing buffered disk reads:  50 MB in  3.01 seconds =  16.61 MB/sec
 ```
 
等多信息可参考[Wiki](https://wiki.archlinux.org/index.php/USB_Storage_Devices#Auto-mounting_with_udev)

## Pacman
ArchLinux的包管理软件是pacman，类似apt-get, yum等，这里有所有的包：[http://archlinuxarm.org/packages](http://archlinuxarm.org/packages)

[使用方法](https://wiki.archlinux.org/index.php/Pacman)

1. pacman -Syu && sync #更新整个系统，**新安装好要运行一次**
2. pacman -S gcc make git #安装gcc, make等，作为程序员必须的
3. pacman -R package_name --nosave #删除干净某个包
3. pacman -Scc #完全清理包缓存

## UnixBench

了解一下性能基准测试非常有必要。

```
curl http://byte-unixbench.googlecode.com/files/unixbench-5.1.2.tar.gz -o unixbench-5.1.2.tar.gz
tar zxvf unixbench-5.1.2.tar.gz
cd unixbench-5.1.2
make
./Run
```
如果没有X，要在Makefile里注释掉X的测试，结果如下：

```
========================================================================
   BYTE UNIX Benchmarks (Version 5.1.2)

   System: raspberrypi: GNU/Linux
   OS: GNU/Linux -- 3.2.27+ -- #250 PREEMPT Thu Oct 18 19:03:02 BST 2012
   Machine: armv6l (unknown)
   Language: en_US.utf8 (charmap="ANSI_X3.4-1968", collate="ANSI_X3.4-1968")
   23:12:18 up 63 days, 22:35,  2 users,  load average: 0.27, 0.28, 0.23; runlevel 2

------------------------------------------------------------------------
Benchmark Run: Sat Mar 09 2013 23:12:18 - 23:41:57
0 CPUs in system; running 1 parallel copy of tests

Dhrystone 2 using register variables        1686980.7 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                      269.9 MWIPS (10.0 s, 7 samples)
Execl Throughput                                256.8 lps   (29.7 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks         43489.0 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks           14568.0 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks         96518.7 KBps  (30.0 s, 2 samples)
Pipe Throughput                              172158.1 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                  24098.7 lps   (10.0 s, 7 samples)
Process Creation                                772.2 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                    462.6 lpm   (60.1 s, 2 samples)
Shell Scripts (8 concurrent)                     59.0 lpm   (60.5 s, 2 samples)
System Call Overhead                         396466.7 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0    1686980.7    144.6
Double-Precision Whetstone                       55.0        269.9     49.1
Execl Throughput                                 43.0        256.8     59.7
File Copy 1024 bufsize 2000 maxblocks          3960.0      43489.0    109.8
File Copy 256 bufsize 500 maxblocks            1655.0      14568.0     88.0
File Copy 4096 bufsize 8000 maxblocks          5800.0      96518.7    166.4
Pipe Throughput                               12440.0     172158.1    138.4
Pipe-based Context Switching                   4000.0      24098.7     60.2
Process Creation                                126.0        772.2     61.3
Shell Scripts (1 concurrent)                     42.4        462.6    109.1
Shell Scripts (8 concurrent)                      6.0         59.0     98.3
System Call Overhead                          15000.0     396466.7    264.3
                                                                   ========
System Benchmarks Index Score                                          99.9


========================================================================
   BYTE UNIX Bench  marks (Version 5.1.2)

   System: raspberrypi2: GNU/Linux
   OS: GNU/Linux -- 3.6.11-8-ARCH+ -- #1 PREEMPT Sat Mar 9 00:38:58 UTC 2013
   Machine: armv6l (unknown)
   Language: en_US.utf8 (charmap="UTF-8", collate="ANSI_X3.4-1968")
   23:11:34 up 40 min,  2 users,  load average: 0.32, 0.56, 0.40; runlevel 5

------------------------------------------------------------------------
Benchmark Run: Sat Mar 09 2013 23:11:34 - 23:40:13
0 CPUs in system; running 1 parallel copy of tests

Dhrystone 2 using register variables        1686859.5 lps   (10.1 s, 7 samples)
Double-Precision Whetstone                      240.0 MWIPS (10.0 s, 7 samples)
Execl Throughput                                235.9 lps   (29.8 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks         36862.7 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks           11351.7 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks         79915.7 KBps  (30.0 s, 2 samples)
Pipe Throughput                              127650.5 lps   (10.1 s, 7 samples)
Pipe-based Context Switching                  18840.9 lps   (10.1 s, 7 samples)
Process Creation                                779.2 lps   (30.1 s, 2 samples)
Shell Scripts (1 concurrent)                    193.1 lpm   (60.3 s, 2 samples)
Shell Scripts (8 concurrent)                     26.7 lpm   (60.7 s, 2 samples)
System Call Overhead                         314659.8 lps   (10.1 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0    1686859.5    144.5
Double-Precision Whetstone                       55.0        240.0     43.6
Execl Throughput                                 43.0        235.9     54.9
File Copy 1024 bufsize 2000 maxblocks          3960.0      36862.7     93.1
File Copy 256 bufsize 500 maxblocks            1655.0      11351.7     68.6
File Copy 4096 bufsize 8000 maxblocks          5800.0      79915.7    137.8
Pipe Throughput                               12440.0     127650.5    102.6
Pipe-based Context Switching                   4000.0      18840.9     47.1
Process Creation                                126.0        779.2     61.8
Shell Scripts (1 concurrent)                     42.4        193.1     45.6
Shell Scripts (8 concurrent)                      6.0         26.7     44.5
System Call Overhead                          15000.0     314659.8    209.8
                                                                   ========
System Benchmarks Index Score                                          76.3

```
看上去ArchLinux性能差了一节，看来官方推荐Raspian确实做了不少优化，我觉得介绍一下系统方面的优化，也是非常不错的内容。
                                                                   

## 无线网络
我买的是基于RT5370芯片组的腾达W311MI，Raspberry Pi支持的很好。

1. 确认系统已经识别USB网卡，如下`RT5370 Wireless Adapter`就代表已经识别成功

    ```
    [root@raspberrypi2 ~]# lsusb
    Bus 001 Device 002: ID 0424:9512 Standard Microsystems Corp. LAN9500 Ethernet 10/100 Adapter
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. 
    Bus 001 Device 004: ID 148f:5370 Ralink Technology, Corp. RT5370 Wireless Adapter
    ```    

2. 安装无线工具

    ```
    pacman -S wireless_tools
    ```
    
3. 设置开机启动无线网络
    
    使用`wifi-menu`手动连上wifi ap，可以连多个，相应的输入会保存在：/etc/network.d/，在下面的文件里输入相应的文件名
    
    修改/etc/conf.d/netcfg
    
    ```
    DHCP_TIMEOUT=30 
    AUTO_PROFILES=("wlan0-Hugo2" "wlan0-hugo")
    ```
    
    如果是隐藏SSID的要加一行"HIDDEN=YES"
    
    执行一下命令在重启时自动连上wifi
    ```
    systemctl enable net-auto-wireless
    ```

4. 有条件的可以在路由器里设置好根据MAC地址总是分配同一个ip给Pi，这样就可以拔掉网线的束缚了~

5. 测试了断开后可以自动重连

6. 用scp测试从Mac通过无线传大文件到Raspberry Pi，传输速度只有1.6MB/s，如果通过网线传则有4MB/s

#
# Samba

1. 安装相关包： `pacman -S samba` 
2. 生成一个配置文件： `cp /etc/samba/smb.conf.default /etc/samba/smb.conf` 
3. 加到启动脚本里： `systemctl enable smbd.service`
4. 增加一个samba用户： `smbpasswd -a hugo`
