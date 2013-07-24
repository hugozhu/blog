---
date: 2013-07-21
layout: post
title: 使用夏普GP2Y1010AU0F灰尘传感器检测空气质量
description: Sharp GP2Y1010AU0F Optical Dust Sensor
categories:
- Blog
tags:
- Arduino

---

{:toc}

夏普[GP2Y1010AU0F](http://item.taobao.com/item.htm?spm=a230r.1.14.1.X4yMiN&id=25584528001&_u=oqa3375a)灰尘传感器价格较便宜，能检测出室内空气中的灰尘和烟尘含量。另外还有韩国SYHITECH生产的[DSM501A](http://item.taobao.com/item.htm?spm=a230r.1.14.9.NNhRMO&id=15543884159)粉尘传感器也有类似功能。


# 检测原理

其原理如下图，传感器中心有个洞可以让空气自由流过，定向发射LED光，通过检测经过空气中灰尘折射过后的光线来判断灰尘的含量。

<img src="https://www.evernote.com/shard/s26/sh/50d1faab-e7fc-45c6-9a0f-2b5f6743ce7f/4422cbee4a324b25d1ebaabba3eb350f/deep/0/www.beck-elektronik.de/fileadmin/templates/beck_folder/opto/sensor/sharp/an-gp2y1010au.pdf.png" width="500"/>

# 电路图

<img src="https://www.evernote.com/shard/s26/sh/d34f08d5-0a93-44cc-b360-0b0686efa11d/e8548764c41977130e79b645e8b82cc3/deep/0/www.beck-elektronik.de/fileadmin/templates/beck_folder/opto/sensor/sharp/an-gp2y1010au.pdf.png" width="500"/>

<img src="http://ww1.sinaimg.cn/bmiddle/6bc40342jw1e6x61e41ykj218g0xcamc.jpg" width="500"/>
因为数据是通过pin 5的电压模拟信号输出的，而树莓派的引脚不支持模拟信号直接读取（需要增加数模转换芯片），所以先用Arduino来实验。

# Arduino 代码

<img src="https://www.evernote.com/shard/s26/sh/5159a7b8-8e50-49ac-9a5f-63deb93c539d/5cee487cce5d3b3cbc530bc9e979908b/deep/0/%E6%88%91%E7%9A%84%E5%BE%AE%E5%8D%9A%20%E6%96%B0%E6%B5%AA%E5%BE%AE%E5%8D%9A-%E9%9A%8F%E6%97%B6%E9%9A%8F%E5%9C%B0%E5%88%86%E4%BA%AB%E8%BA%AB%E8%BE%B9%E7%9A%84%E6%96%B0%E9%B2%9C%E4%BA%8B%E5%84%BF.png" width="400"/>

根据电路图， 把Arduino和传感器连接起来：

1. Sharp pin 1 (V-LED)   => 5V 串联1个[150欧姆的电阻](http://detail.tmall.com/item.htm?id=13437610350&spm=a230r.1.14.1.BiUeau&ad_id=&am_id=&cm_id=140105335569ed55e27b&pm_id=)（最好在电阻一侧和GND之间再串联一个[220uf的电容](http://detail.tmall.com/item.htm?id=13661538754&spm=a230r.1.14.17.KlljiT&ad_id=&am_id=&cm_id=140105335569ed55e27b&pm_id=)）
2. Sharp pin 2 (LED-GND) => GND
3. Sharp pin 3 (LED)     => Arduino PIN 2 （开关LED）
4. Sharp pin 4 (S-GND)   => GND
5. Sharp pin 5 (Vo)      => Arduino A0 pin （空气质量数据通过电压模拟信号输出）
6. Sharp pin 6 (Vcc)     => 5V


```
/*
 Interface to Sharp GP2Y1010AU0F Particle Sensor
 Program by Christopher Nafis 
 Written April 2012
 
 http://www.sparkfun.com/datasheets/Sensors/gp2y1010au_e.pdf
 http://sensorapp.net/?p=479
 
 Sharp pin 1 (V-LED)   => 5V (connected to 150ohm resister)
 Sharp pin 2 (LED-GND) => Arduino GND pin
 Sharp pin 3 (LED)     => Arduino pin 2
 Sharp pin 4 (S-GND)   => Arduino GND pin
 Sharp pin 5 (Vo)      => Arduino A0 pin
 Sharp pin 6 (Vcc)     => 5V
 */
#include <SPI.h>
#include <stdlib.h>

int dustPin=0;
int ledPower=2;
int delayTime=280;
int delayTime2=40;
float offTime=9680;

int dustVal=0;
int i=0;
float ppm=0;
char s[32];
float voltage = 0;
float dustdensity = 0;
float ppmpercf = 0;

void setup(){
  Serial.begin(9600);
  pinMode(ledPower,OUTPUT);

  // give the ethernet module time to boot up:
  delay(1000);

  i=0;
  ppm =0;
}

void loop(){
  i=i+1;
  digitalWrite(ledPower,LOW); // power on the LED
  delayMicroseconds(delayTime);
  dustVal=analogRead(dustPin); // read the dust value
  ppm = ppm+dustVal;
  delayMicroseconds(delayTime2);
  digitalWrite(ledPower,HIGH); // turn the LED off
  delayMicroseconds(offTime);

  voltage = ppm/i*0.0049;
  dustdensity = 0.17*voltage-0.1;
  ppmpercf = (voltage-0.0256)*120000;
  if (ppmpercf < 0)
    ppmpercf = 0;
  if (dustdensity < 0 )
    dustdensity = 0;
  if (dustdensity > 0.5)
    dustdensity = 0.5;
  String dataString = "";
  dataString += dtostrf(voltage, 9, 4, s);
  dataString += ",";
  dataString += dtostrf(dustdensity, 5, 2, s);
  dataString += ",";
  dataString += dtostrf(ppmpercf, 8, 0, s);
  i=0;
  ppm=0;
  Serial.println(dataString);
  delay(1000);
}
```

把传感器和Ardiuno连接好后，可以连续打印出传感器的输出电压值。输出电压大小和灰尘含量的曲线入下图：

<img src="https://www.evernote.com/shard/s26/sh/d9213b42-a489-4b10-a6a6-24b170f5bcab/26d6af8a0fb1c572c7f2c231bb376255/deep/0/%E5%85%B3%E4%BA%8EGP2Y1010AU0F%C2%A0SHARP%E4%BC%A0%E6%84%9F%E5%99%A8%E4%BD%BF%E7%94%A8_dizhuwa_%E6%96%B0%E6%B5%AA%E5%8D%9A%E5%AE%A2.png" width="400"/>

通过电压的波形还可以判断是烟还是尘呢。。。
<img src="https://www.evernote.com/shard/s26/sh/398f3903-7aff-444c-847e-3520e049d142/d7f12635352a30d50bdc4c30b2930367/deep/0/Dust%C2%A0Sensor%C2%A0GP2Y1010AU%C2%A0Application%C2%A0Note_PM2.5%E4%BC%A0%E6%84%9F%E5%99%A8_%E6%96%B0%E6%B5%AA%E5%8D%9A%E5%AE%A2.png" width="500"/>


# 参考链接
1. [Sharp GP2Y1010AU0F Data Sheet](https://www.sparkfun.com/datasheets/Sensors/gp2y1010au_e.pdf)
2. [Sharp GP2Y1010AU0F Application Note](http://www.beck-elektronik.de/fileadmin/templates/beck_folder/opto/sensor/sharp/an-gp2y1010au.pdf)
3. http://www.howmuchsnow.com/arduino/airquality/
