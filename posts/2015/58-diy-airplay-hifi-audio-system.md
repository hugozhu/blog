---
date: 2015-03-15
layout: post
title:  自己搭建Airplay高清音乐播放系统
description: DIY airplay HIFI audio system
categories:
- Blog
tags:
- Raspberry Pi
- pcDunio Acadia


---

{:toc}

先看一下在iPhone 6 plus上用虾米播放高音质音乐的效果：

![Airplay](https://www.evernote.com/shard/s26/sh/5d6ddb93-220e-4c1d-a042-87a8835199dc/5e8345944ca4f948ee84636dacc77358/deep/0/77fa703f-71bc-40ac-8f65-861689fc2085-720-1,280-pixels.png)

苹果的[Airplay](http://en.wikipedia.org/wiki/AirPlay)协议是通过基于[RSTP](http://en.wikipedia.org/wiki/Real_Time_Streaming_Protocol)（Real Time Streaming Protocol）协议通过UDP传输的无损编码音频流([Apple Lossless codec](http://en.wikipedia.org/wiki/Apple_Lossless)，音频流本身经过了AES加密，私钥是不公开的，这样只有苹果和其合作伙伴才能使用这个协议。但是2004年有人通过逆向工程获得了私钥并将其公开，这样我们也可以自己搭建低成本高音质的基于Airplay的音乐系统。在iPhone成为街机的时代，每个人的手机里有很多喜欢的音乐，如果在家里可以通过无线网络在音响系统里播放会方便很多。

# 硬件
1. pcDunio Acadia 或 Raspberry Pi 一只
2. 3.5mm音频线一根，类似[这种](http://detail.tmall.com/item.htm?spm=a230r.1.14.20.nvOtPL&id=27440984038&ad_id=&am_id=&cm_id=140105335569ed55e27b&pm_id=&abbucket=18)
3. 有源音箱一对
4. 网线一根或USB无线网卡一只

# 音频芯片
好的音质需要好的音源，无损音乐加好的前端输出是必不可少的。
pcDuino Acadia集成的音频芯片是业界领先的英国[Wolfson（欧胜微电子）](http://en.wikipedia.org/wiki/Wolfson_Microelectronics)为高清音频设计的[WM8962](http://www.cirrus.com/en/products/wm8962-62b.html) 。树莓派集成的音频芯片则是由美国Broadcom（博通）封装在主芯片[BCM2835](http://www.broadcom.com/products/BCM2835)里通过PWM (pulse-width modulation) 提供的，比较简单音质一般，达不到高清音频的要求。

![raspberry pi audio](http://i2.wp.com/www.crazy-audio.com/wp-content/uploads/2013/11/onboard.png?resize=300%2C180) 

<br/>
下面是树莓派音频口播放1kHz正弦信号的输出波形，可以看出来波形并不好。
![raspberry pi audio2](http://i1.wp.com/www.crazy-audio.com/wp-content/uploads/2013/11/thd_onboard.png?resize=300%2C213)

正是因为板载音频质量差强人意，Wolfson也专为树莓派设计了基于[WM5102](http://www.cirrus.com/en/products/wm5102.html) codec芯片的[Wolfson Audio Card](http://www.adafruit.com/product/1761) （售价高达$34.95，性价比不高，这块芯片也用在了魅族MX3上）。
![Wolfson Audio Card](https://www.evernote.com/shard/s26/sh/7321c82f-96ad-42aa-9171-f10d0da456c0/c65f41888045af7bf1d879320ee3e171/deep/0/1761-00.jpg-970-728-pixels.png)

更好的方案是使用USB声卡，如淘宝上可以购买的基于德州仪器的[PCM2704](http://www.ti.com/product/pcm2704)的[USB声卡](http://s.taobao.com/search?q=PCM2704&commend=all&ssid=s5-e&search_type=item&sourceId=tb.index&spm=1.7274553.1997520841.1&initiative_id=tbindexz_20150315)，40多人民币。

![PCM2704 USB DAC](http://g.search1.alicdn.com/img/bao/uploaded/i4/i1/11699024068466586/T144m3XyFcXXXXXXXX_!!0-item_pic.jpg_250x250.jpg)

综上，如果想DIY自己的高清音乐播放系统，只有树莓派还是不够的，可以考虑用pcDunio Acadia或额外购买USB DAC。

# 声音测试和调节

## 声音测试
首先需要用音频线把板子和有源音箱连接起来，如果线材或音箱质量不够好，会立刻听到背景噪音，实测树莓派的背景噪音比pcDuino Acadia要大一些。

注：普通用户不一定有权限使用系统音频设备，所以以下测试需要root或audio group里的用户。

首先可以用`speaker-test`命令测试是否能出声。
```
hugo@raspberrypi2 ~ $ sudo speaker-test 
speaker-test 1.0.25

Playback device is default
Stream parameters are 48000Hz, S16_LE, 1 channels
Using 16 octaves of pink noise
Rate set to 48000Hz (requested 48000Hz)
Buffer size range from 512 to 32768
Period size range from 512 to 32768
Using max buffer size 32768
Periods = 4
was set period_size = 8192
was set buffer_size = 32768
 0 - Front Left
Time per period = 2.234240
 0 - Front Left
Time per period = 2.899088
 0 - Front Left
```
如果听不到声音，可以检查一下连接和系统设置：

树莓派的命令输出：

```
hugo@raspberrypi2 ~ $ sudo aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: ALSA [bcm2835 ALSA], device 0: bcm2835 ALSA [bcm2835 ALSA]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 0: ALSA [bcm2835 ALSA], device 1: bcm2835 ALSA [bcm2835 IEC958/HDMI]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

pcDunio Acadia的命令输出：

```
    Hardware device with all software conversions
hugo@Acadia ~ $ sudo aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: wm8962audio [wm8962-audio], device 0: HiFi wm8962-0 []
  Subdevices: 0/1
  Subdevice #0: subdevice #0
card 1: imxhdmisoc [imx-hdmi-soc], device 0: IMX HDMI TX mxc-hdmi-soc-0 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
  ```
  
如果你安装了USB声卡，还需要设置系统把USB声卡设置成缺省声卡。

`sudo vi /etc/modprobe.d/alsa-base.conf`

修改

`options snd-usb-audio index=-2`

为

```
#options snd-usb-audio index=-2  
options snd-usb-audio nrpacks=
```

有些系统可能还需执行下一步（注：树莓派2不需要）

`sudo vi /usr/share/alsa/alsa.conf`
把`pcm.front cards.pcm.front` 替换成`pcm.front cards.pcm.default`

## 测试音乐播放
下载一个mp3文件在pcDunio或树莓派上，然后安装`mpg321`

```
sudo apt-get install mpg321
```
执行`mpg321 example.mp3`就可以测试音乐播放了

## 音量调节

可以执行`sudo alsamixer`来设置音量，执行`sudo alsactl store 0`恢复出厂设置。

对比一下可以看到pcDunio Acadia的设置项要丰富多了，除了Headphone这项可以提高音量(5dB)外，Digital这项的增益调节也可以大幅提高音量（23.25dB）。推力方面也相对好多了，可以直推一些好耳机。

![pi](https://www.evernote.com/shard/s26/sh/924d1a7f-d4fb-4cb6-aea9-4b45843a7226/439c2ae62288609058555c06006ec99b/deep/0/hugozhu---hugo@raspberrypi2------ssh---65-27.png)

![pcDunio](https://www.evernote.com/shard/s26/sh/5f01fea9-0726-45ac-a1d0-7ef508862487/f0a830f6c56b40744a1ab62a3623fdf4/deep/0/workapp---hugo@Acadia------ssh---98-35.png)

## Airplay设置

这里我们使用开源的Airplay解码软件`Shairport`。

## 安装依赖库

```
sudo apt-get install git libao-dev libssl-dev libcrypt-openssl-rsa-perl libio-socket-inet6-perl libwww-perl avahi-utils libmodule-build-perl libasound2-dev libpulse-dev
```
 

## 下载代码，编译安装

```
root@Acadia ~ $ git clone https://github.com/abrasive/shairport.git  
root@Acadia ~ $ cd shairport  
root@Acadia ~ $ ./configure  
Configuring Shairport  
OpenSSL found  
libao found  
PulseAudio found  
ALSA found  
Avahi client found  
getopt.h found  
CFLAGS: -D_REENTRANT -I/usr/include/alsa -D_REENTRANT  
LDFLAGS: -lm -lpthread -lssl -lcrypto -lao -lpulse-simple -lpulse -lasound -lavahi-common -lavahi-client  
Configure successful. You may now build with 'make'  
root@Acadia ~/shairport $ make  
```

pcDunio Acadia上还需要执行这一步：

```
root@Acadia ~ $ git clone https://github.com/njh/perl-net-sdp.git perl-net-sdp  
root@Acadia  ~ $ cd perl-net-sdp  
root@Acadia  ~/perl-net-sdp $ perl Build.PL  
root@Acadia  ~/perl-net-sdp $ sudo ./Build  
root@Acadia  ~/perl-net-sdp $ sudo ./Build test  
root@Acadia  ~/perl-net-sdp $ sudo ./Build install  
root@Acadia  ~/perl-net-sdp $ cd ..
```

## 运行Shairport
运行非常简单：
```
sudo shairport -a "Acadia"
```
执行完后拿起你的iPhone，虾米，QQ音乐或百度音乐App都支持Airplay了，在播放界面上选择输出到`Acadia`上就可以开始欣赏Airplay啦。

#  参考文章
1. http://www.crazy-audio.com/2013/11/quality-of-the-raspberry-pi-onboard-sound/
2. http://drewlustro.com/hi-fi-audio-via-airplay-on-raspberry-pi/
3. http://raspberrypihq.com/how-to-turn-your-raspberry-pi-into-a-airplay-receiver-to-stream-music-from-your-iphone/
