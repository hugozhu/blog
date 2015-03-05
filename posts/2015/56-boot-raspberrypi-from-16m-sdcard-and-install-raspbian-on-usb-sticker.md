---
date: 2015-03-05
layout: post
title: 用16M的SD卡启动树莓派，在U盘上安装和运行Linux
description: Boot Raspberry Pi from 16M SDCard
categories:
- Blog
tags:
- Raspberry Pi


---

{:toc}

树莓派官方的操作系统Raspbian最少需要4G的SDCard，如果你恰好有比较小的SD卡怎么办呢？设计上树莓派只能从SD卡引导启动，但我们可以在SD卡上装一个最小的引导系统，然后把树莓派引导到U盘上启动。SD卡连续运行，读写比较频繁也容易损坏（我已经坏掉3张了。。。），相比之下U盘价格便宜些，读写速度可以比SD卡还高。

# 制作引导SD卡

这个引导系统只需要16M容量的SD卡，先把SD卡格式化成FAT（windows），然后拷贝下面的文件到根目录。

```
git clone https://github.com/hugozhu/mini_raspbian_boot
```

修改 `cmdline.txt` 中的 `root=/dev/mmcblk0p2` 为 `root=/dev/sda2` 以指定用U盘启动

```
dwc_otg.lpm_enable=0 console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait cgroup_enable=memory
```

# 安装操作系统Raspbian到U盘
方法和安装到SD卡一样，在Mac上使用 `dd` 命令安装，在Windows上可以用win32diskimager：

```
sudo dd bs=1m if=2015-01-31-raspbian.img of=/dev/disk4 #/dev/disk4
```

# 扩展分区
如果你的U盘容量大于4G，那么上一步安装完后，你的U盘只能看到4G空间，大容量的SD卡的这个问题可以通过自带的`raspi-config`命令来解决。
Linux下可以用`fdisk`来解决，我的方法是把该U盘插到已经正常启动的树莓派的USB口上，正常识别后，执行`sudo fdisk /dev/sda`

1. 按`p`后打印出分区表：

```
/dev/sda1            8192      122879       57344    c  W95 FAT32 (LBA)
/dev/sda2          122880     6399999     3138560   83  Linux
```

2. 记住/dev/sda2的起始点`122880'，然后按`d`，删除掉二个分区，接着按`n`重新创建一个分区，起始点就设置为`122880`，结束点可以设置为最大

3. 最后按`w`保存新的分区表

插上引导SD卡和系统U盘后树莓派就可以愉快的运行起来了，如果U盘的速度足够快，你会发现系统还变快了。

# 参考文章
1. http://www.raspberrypi.org/boot-from-a-16mb-sd-card/
