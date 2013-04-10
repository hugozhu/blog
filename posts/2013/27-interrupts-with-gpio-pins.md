---
date: 2013-04-08
layout: post
title: Raspberry Pi的GPIO中断编程
description: Interrupts with Raspberry Pi GPIO Pins
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

# 背景介绍

树莓派的GPIO引脚不仅可以输出高低电平，也可以当做输入端口（可以想象成键盘输入），当GPIO接入的是高电平，GPIO的值可以认为是1，如果是低电平则是0。如下图所示，可以使用一个Push Button开关按键来控制GPIO 25（BCM Numbering）的高低电平以达到控制的目的。

<img src="https://www.evernote.com/shard/s26/sh/bd720803-1b71-454d-98a2-cf0e907df688/649d6eb30376f8a378ef3e8a7cc3c552/deep/0/Screenshot%204/8/13%2011:25%20PM.jpg?noteKey=649d6eb30376f8a378ef3e8a7cc3c552&suffix=deep%2F0%2FScreenshot+4%2F8%2F13+11%3A25+PM.jpg&noteGuid=bd720803-1b71-454d-98a2-cf0e907df688"/>

GPIO 25和VCC（3.3V）之间通过R1（10K欧姆）和R2（1K欧姆）[上拉电阻](https://zh.wikipedia.org/zh/上拉电阻)相连，当按键未被按下时，GPIO 25上拉到VCC，程序可以读到1，当按键按下时，GPIO 25被下拉电阻R2拉到GND（0V），程序可以读到0。如果不加R1，而GPIO 25不小心被设置成输出低电平时，将直接和VCC相连而造成短路，这样可能会烧掉这个引脚，所以加上限流电阻R1后，即使发生这样的情况，也不会出现短路情况。

# 应用

如果我们需要根据GPIO 25的值来控制树莓派，比如按下按钮时希望点亮某个LED或在液晶上显示当前时间，就需要通过程序来获取状态的变化。

一种常见的做法是在循环里不断读取该引脚的状态，当发生对应的变化的时执行控制逻辑，但显而易见，这种做法很消耗CPU，如果在循环增加`sleep(1000)`这样的调用，又很容易错过按键变化。较好的做法则是通过[中断](http://zh.wikipedia.org/wiki/中断)来实现。

最新的树莓派Raspbian和Arch Linux内核都已经包含了GPIO的中断处理支持。但使用前需要将指定GPIO引脚输出，方法如下：

首先可以通过命令`echo 25 > /sys/class/gpio/export`导出GPIO 25端口，执行成功后在相应的目录下看到以下文件，得益于Linux下**一切都是文件**的设计理念，GPIO的状态可以通过`value`文件来获取，这样就可以利用Linux的poll/epoll来获取`value`文件的变化(这点和Linux高性能网络编程是类似的)。

```
root@raspberrypi2 ~/projects/interrupt_test # ls -l /sys/class/gpio/gpio25/
total 0
-rw-r--r-- 1 root root 4096 Apr  8 23:56 active_low
-rw-r--r-- 1 root root 4096 Apr  8 22:29 direction
-rw-r--r-- 1 root root 4096 Apr  8 22:29 edge
drwxr-xr-x 2 root root    0 Apr  8 23:56 power
lrwxrwxrwx 1 root root    0 Apr  8 23:56 subsystem -> ../../../../class/gpio
-rw-r--r-- 1 root root 4096 Apr  8 22:08 uevent
-rw-r--r-- 1 root root 4096 Apr  8 22:29 value
root@raspberrypi2 ~/projects/interrupt_test # 
```

# wiringPi
[wiringPi](https://projects.drogon.net/raspberry-pi/wiringpi/functions/)库封装了一个简单的接口，传入一个回调函数，当事件发生时传入的函数将被调用。

```
int wiringPiISR (int pin, int edgeType,  void (*function)(void)) ;
```
其最主要的部分的实现代码是：

```
int waitForInterruptSys (int pin, int mS)
{
  int fd, x ;
  uint8_t c ;
  struct pollfd polls ;

  if ((fd = sysFds [pin & 63]) == -1)
    return -2 ;

// Setup poll structure

  polls.fd     = fd ;
  polls.events = POLLPRI ;      // Urgent data!

// Wait for it ...

  x = poll (&polls, 1, mS) ;

// Do a dummy read to clear the interrupt
//      A one character read appars to be enough.

  (void)read (fd, &c, 1) ;

  return x ;
}
```

# 示例代码

```
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <stdlib.h>
#include <wiringPi.h>


// What GPIO input are we using?
//      This is a wiringPi pin number

#define BUTTON_PIN      6

static volatile int globalCounter = 0 ;


void myInterrupt (void)
{
  ++globalCounter ;
}


int main (void)
{
  int myCounter = 0 ;

  if (wiringPiSetup () < 0)
  {
    fprintf (stderr, "Unable to setup wiringPi: %s\n", strerror (errno)) ;
    return 1 ;
  }

  if (wiringPiISR (BUTTON_PIN, INT_EDGE_FALLING, &myInterrupt) < 0)
  {
    fprintf (stderr, "Unable to setup ISR: %s\n", strerror (errno)) ;
    return 1 ;
  }


  for (;;)
  {
    printf ("Waiting ... ") ; fflush (stdout) ;

    while (myCounter == globalCounter)
      delay (100) ;

    printf (" Done. counter: %5d\n", globalCounter) ;
    myCounter = globalCounter ;
  }

  return 0 ;
}
``` 

# RPi.GPIO

Python的PRI.GPIO库也对中断进行了封装，以下是使用例子，差别在于wiringPi支持多线程，允许在等待中断的时候干点别的。

```
#!/usr/bin/env python2.7  
# script by Alex Eames http://RasPi.tv/  
# http://raspi.tv/2013/how-to-use-interrupts-with-python-on-the-raspberry-pi-and-rpi-gpio  
import RPi.GPIO as GPIO  
GPIO.setmode(GPIO.BCM)  
  
# GPIO 25 set up as input. It is pulled up to stop false signals  
GPIO.setup(25, GPIO.IN, pull_up_down=GPIO.PUD_UP)  
  
print "Make sure you have a button connected so that when pressed"  
print "it will connect GPIO port 23 (pin 16) to GND (pin 6)\n"  
raw_input("Press Enter when ready\n>")  
  
print "Waiting for falling edge on port 23"  
# now the program will do nothing until the signal on port 23   
# starts to fall towards zero. This is why we used the pullup  
# to keep the signal high and prevent a false interrupt  
  
print "During this waiting time, your computer is not"   
print "wasting resources by polling for a button press.\n"  
print "Press your button when ready to initiate a falling edge interrupt."  
while 1:
    try:  
        GPIO.wait_for_edge(25, GPIO.FALLING)  
        print "\nFalling edge detected. Now your program can continue with"  
        print "whatever was waiting for a button press."  
    except KeyboardInterrupt:  
        GPIO.cleanup()       # clean up GPIO on CTRL+C exit  
GPIO.cleanup()           # clean up GPIO on normal exit  
```

# 参考链接
1. https://projects.drogon.net/raspberry-pi/wiringpi/functions/
2. http://raspi.tv/2013/how-to-use-interrupts-with-python-on-the-raspberry-pi-and-rpi-gpio