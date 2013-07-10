---
date: 2013-07-11
layout: post
title: 替换树莓派的U盘
description: Replace USB sticker on Raspberry Pi
categories:
- Blog
tags:
- Linux

---

除了SD卡上的存储，树莓派还可以使用U盘来做存储，有时候我们可能需要替换已有的U盘为更大容量的。在Mac上可以采用下面的方法：

1. 备份已有的U盘，把U盘从树莓派上拔下来插在Mac上，找出U盘对应的盘符（下例为`/dev/disk2`）
    ```
    20:51:19 hugozhu-mac-mini ~ $ diskutil list
    /dev/disk0
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:      GUID_partition_scheme                        *500.1 GB   disk0
       1:                        EFI                         209.7 MB   disk0s1
       2:                  Apple_HFS Macintosh HD            499.2 GB   disk0s2
       3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
    /dev/disk2
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:     FDisk_partition_scheme                        *2.1 GB     disk2
    ```

    使用 dd 命令把U盘拷贝到`raspberrypi.img`

    ```
    sudo dd if=/dev/disk2 of=raspberrypi.img conv=notrunc
    ```
2. 从Mac上取下旧U盘，把新的U盘插入同一个USB口，注意新U盘容量要大于旧的

    ```
    sudo dd of=/dev/disk2 if=raspberrypi.img conv=notrunc
    ```
    
3. 把新U盘插入树莓派，并mount上，用以下命令把U盘的多余空间用起来

    ```
    resize2fs /dev/sda
    ```