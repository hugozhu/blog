---
date: 2013-04-08
layout: post
title: 备份Raspberry Pi
description: Backup Raspberry Pi
categories:
- Blog
tags:
- Rasperry Pi

---

树莓派的操作系统安装在SD卡，使用一段时间后还是很有必要备份一下，以防哪天SD卡就坏了。

备份的目的地最方便的还是使用网络存储，我使用的是西部数据的[MyBooklive](http://detail.tmall.com/item.htm?spm=a220m.1000858.1000725.1.Cz5Mlq&id=13865367896&is_b=1&cat_id=50099232&q=mybooklive&rn=638aa11bdda81f8d589bb0e052c57187)3T网络硬盘。挺不错的一个产品，功能基本满足我的需求。

准备好备份目标盘，将Nas的备份目录mount到树莓派:

```
mkdir /mnt/backup
mount -t cifs //mybooklive/Public/Backup /mnt/backup -o guest
```

# 完整备份

确定相应的SD卡设备ID

```
root@raspberrypi2 ~/bin # fdisk -l

Disk /dev/mmcblk0: 1973 MB, 1973420032 bytes, 3854336 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0004f23a

        Device Boot      Start         End      Blocks   Id  System
/dev/mmcblk0p1   *        2048      186367       92160    c  W95 FAT32 (LBA)
/dev/mmcblk0p2          186368     3667967     1740800   83  Linux

Disk /dev/sda: 2107 MB, 2107637760 bytes, 4116480 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

执行备份

```
cd /mnt/backup
dd if=/dev/mmcblk0 of=raspberrypi2.img bs=2M
```

# 增量备份
增量备份最简单的方法是用rsync，每天定时将指定目录下变化的文件保存到备份目录，方法如下：

```
root@raspberrypi:~# crontab -l
0 3 * * * /root/bin/backup.sh > /var/logs/backup.log 2>&1


root@raspberrypi:~# cat /root/bin/backup.sh 
#!/bin/sh
mount /mnt/backup
sleep 3
rsync -v -a --delete --size-only -O --no-t --no-o --no-p --no-g /var/www /mnt/backup/raspberrypi/
rsync -v -a --delete --size-only -O --no-t --no-o --no-p --no-g /home/hugo /mnt/backup/raspberrypi/
rsync -v -a --delete --size-only -O --no-l --no-t --no-o --no-p --no-g /etc /mnt/backup/raspberrypi/

```