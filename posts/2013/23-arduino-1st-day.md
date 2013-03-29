---
date: 2013-03-28
layout: post
title: Arduino初试
description: Arduino first day
categories:
- Blog
tags:
- Arduino

---

今天拿到一块Arduino UNO R3板，迫不及待就开始试用了。相比Raspberry Pi是一个全能的电脑，Arduino则是个硬件开源的单片机，因为开源，资料和配件网上就很很多了，也就容易让初学者上手了。

Arduino特点:

1. 开源，硬件标准化，配套传感器等模块很多；
2. 结构简单
3. 实时系统，稳定，启动只要0.5秒

# Arduino IDE
[下载Arduino IDE](http://arduino.cc/en/Main/Software)


# 上电测试
用USB线接在电脑USB口，然后在GND和PIN 13上插一个二极管，注意二极管正极插在PIN 13上, 如下图：
<img src="http://ww1.sinaimg.cn/bmiddle/6bc40342jw1e35tmsg0tjj.jpg"/>

(注：还应该串联一个300欧姆的限流电阻才保险！)

# 上传代码
在Arduino IDE编辑好下面的代码，然后点Upload后就会运行了，会看到LED一闪一闪。

```
/*
  Blink
  Turns on an LED on for one second, then off for one second, repeatedly.
 
  This example code is in the public domain.
 */
 
// Pin 13 has an LED connected on most Arduino boards.
// give it a name:
int led = 13;

// the setup routine runs once when you press reset:
void setup() {                
  // initialize the digital pin as an output.
  pinMode(led, OUTPUT);     
}

// the loop routine runs over and over again forever:
void loop() {
  digitalWrite(led, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);               // wait for a second
  digitalWrite(led, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);               // wait for a second
}
```

# 参考链接

1. http://arduino.cc/en/Tutorial/Blink
