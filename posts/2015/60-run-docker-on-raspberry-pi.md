---
date: 2015-04-12
layout: post
title:  在树莓派上运行Docker
description: Run docker on Raspberry Pi
categories:
- Blog
tags:
- Raspberry Pi
- docker


---

{:toc}

# Docker

Docker是目前非常流行的代码运行容器，操作系统虚拟化，运维自动化架构和开源的解决方案。
Docker的基础是Linux容器（LXC）技术，相比传统的VM虚拟化技术, LXC更轻量，性能更好。
Docker采用Golang语言开发，在LXC基础上Docker进行了封装，简化了容器的管理。
Docker还提供了一个标准(Dockerile)来实现软件部署环境代码化，全球的开发和运维工程师可以通过官方的Docker Hub仓库分享自己创建的镜像，使用者则可以快速的把系统和应用部署到自己的环境。


# 树莓派二代
树莓派二代的CPU有4核，运行速度是第一代的6倍，通过Docker快速部署开发环境，开发应用，再把开发好的系统通过镜像分享和发布出去也是非常有意义的事。因为树莓派的官方操作系统Raspbian并不支持Docker，本文主要介绍下如何在树莓派二代上运行Docker的几种方法。

## Arch Linux ARM
[Arch Linux ARM](http://archlinuxarm.org) 是由开源社区维护专为ARMv6（如树莓派一代）和 ARMv7（如树莓派二代, pcDuino3）等嵌入式硬件提供内核及软件支持的Linux发行版本。

在树莓派上安装Arch Linux和Raspbian略有不同，详细步骤可以看[参考链接4](http://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2)。
总得来说是你需要在SD卡上分两个区并格式化，一个是FAT 32(LBA)格式的引导分区(/boot)，另一个是系统根分区（/root）。然后把下载下来的文件解开来复制到这两个分区就可以了。

你也可以只在SD卡上放/boot分区（这样可以用较小的如1G的SD卡引导树莓派），在U盘上放根分区，通过USB扩展存储可以允许你存放很多的镜像文件。

安装好Arch Linux后，参考这篇[文章](http://hugozhu.myalert.info/2013/03/09/setup-archliunx-on-raspberry-pi.html)做完基础设置。


### 安装docker

```
pacman -S docker
```
目前安装好的版本是1.5


### 开机自动docker

```
systemctl enable docker
```

### 把登录用户加到docker组
这样不需要root也能执行docker了

```
gpasswd -a <your_login> docker
```

### 执行docker
Docker Hub上搜索rpi已经可以找到不少适合Raspberry Pi运行的镜像了，热心网友都是棒棒哒。

```
[hugo@alarmpi ~]$ docker -v      
Docker version 1.5.0, build a8a31e

[hugo@alarmpi ~]$ docker search rpi
NAME                          DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
resin/rpi-raspbian            Base image for the Raspberry Pi. Contains ...   55                   
sdhibit/rpi-raspbian          Base Raspbian image for ARMv6 boards. Imag...   9                    [OK]
hypriot/rpi-node              RPi-compatible Docker Image with Node.js        8                    
yyolk/rpi-archlinuxarm        Arch Linux Arm base image, built daily w/ ...   7                    
hypriot/rpi-iojs              RPi-compatible Docker Image with io.js          7                    
hypriot/rpi-python            RPi-compatible Docker Image with Python         6                    
hypriot/rpi-java              RPi-compatible Docker Image with Java           5                    
hypriot/rpi-crate             RPi-compatible Docker Image with Crate.io       3                    
akkerman/rpi-nginx            rpi build of nginx                              3                    
jprjr/distcc-rpi                                                              3                    [OK]
resin/rpi-node                Node.js is a JavaScript-based platform for...   1                    
ontouchstart/rpi-ruby         ontouchstart/rpi-ruby                           0                    
ontouchstart/rpi-base         ontouchstart/rpi-base                           0                    
ontouchstart/rpi-redis        ontouchstart/rpi-redis                          0                    
ontouchstart/rpi-go           ontouchstart/rpi-go                             0                    
philipz/rpi-raspbian                                                          0                    [OK]
maiome/rpi-kernel-build       A simple container that makes rebuilding a...   0                    [OK]
dcarley/golang-rpi                                                            0                    [OK]
philipz/rpi-hub-test                                                          0                    [OK]
rpietzsch/paperwork                                                           0                    [OK]
rpietzsch/facete2-docker                                                      0                    [OK]
jakobwesthoff/rpi-cc                                                          0                    [OK]
danielfrg/rpi2xc                                                              0                    [OK]
rpietzsch/bww-personensuche                                                   0                    [OK]
randyp/rpi-archlinuxarm       Adding extra to yyolk/rpi-archlinuxarm...       0                    

```

# HypriotOS

如果你不习惯Arch Linux或者想用docker 1.6，下面这个方法是你的菜：

按照 [http://blog.hypriot.com/post/hypriotos-back-again-with-docker-on-arm/](http://blog.hypriot.com/post/hypriotos-back-again-with-docker-on-arm/) 教程安装就好了。


# Resin.io
这是一家研究在树莓派上运行和分发应用较早的公司。

这里有详细的教程 : [http://docs.resin.io/#/pages/installing/gettingStarted.md](http://docs.resin.io/#/pages/installing/gettingStarted.md)


# 参考链接
1. http://dockerpool.com/static/books/docker_practice/introduction/what.html
2. https://resin.io/blog/docker-on-raspberry-pi/
3. http://blog.hypriot.com/post/hypriotos-back-again-with-docker-on-arm/
4. http://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2