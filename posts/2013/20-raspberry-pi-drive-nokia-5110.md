---
date: 2013-03-24
layout: post
title: 升级版电子钟 - Raspberry Pi和Nokia 5110
description: Nokia 5110 
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}


Nokia 5110屏比前面介绍过的[1602液晶屏](http://hugozhu.myalert.info/2013/03/23/19-raspberry-pi-drive-1602-lcd.html)功能好很多，淘宝上买价格相差不大（二手5110 12块左右, 全新1602 8块左右），Nokia 5110最少只需要占用4个GPIO引脚：
1. 带蓝色背光
2. 使用Philips PCD8544 LCD控制器（通过SPI接口）
3. 84x84点阵，可显示100多个字符

# 硬件准备
1. [树莓派](http://detail.tmall.com/item.htm?spm=a220m.1000858.1000725.6.nGxekg&id=17337394004&is_b=1&cat_id=2&q=%CA%F7%DD%AE%C5%C9&rn=4004716f9ba818c1d69b5eb7818891b5)
2. [Nokia 5110 拆机屏](http://item.taobao.com/item.htm?id=3125173573&ali_trackid=2:mm_12926928_3484851_11423954:1364118594_4k1_653822857&clk1=0b755dfca67112cd1b605914b40146e7&spm=a230z.1.5634029.2.foR1Yu) 或 [全新的?](http://item.taobao.com/item.htm?spm=a230r.1.10.154.SdjftL&id=13361097288&_u=oqa31997) 注意不要买裸屏，需要带电路板的
3. [杜邦线](http://list.tmall.com/search_product.htm?q=%B6%C5%B0%EE%CF%DF&user_action=initiative&at_topsearch=1&sort=st&type=p&cat=&style=) 母对母8条
4. 8P排针 用来焊接5110屏幕PCB板
5. [电烙铁](http://detail.tmall.com/item.htm?id=22096296909&ali_trackid=2:mm_12926928_3484851_11423954:1364118800_4k4_1229281314&clk1=c7335b2dcbad93f47eaf5cd4d1cc140b&spm=a230z.1.5634029.66.6xxdfN)


# 电路
5110电路板有8个引脚，使用排针（如下图）将其焊上，方便后面用杜邦线连接，如果不会焊也可以买焊接好的。

<img src="http://img01.taobaocdn.com/bao/uploaded/i1/13130028971464406/T1v5oAXd0XXXXXXXXX_!!0-item_pic.jpg_600x600.jpg"/>


1. RST —— 复位 接GPIO 0
2. CE  —— 片选 接GPIO 1 或 不接
3. DC  —— 数据/指令选择 接GPIO 2
4. DIN —— 串行数据线 接GPIO 3
5. CLK —— 串行时钟线 接GPIO 5 （因为我的GPIO 4已经接了一个DHT11传感器）
6. VCC —— 电源输入 接3.3v
7. BL  —— 背光控制端 接3.3v
8. GND —— 地线 接地

PS. 编号规范看[这里](http://hugozhu.myalert.info/2013/03/22/19-raspberry-pi-gpio-port-naming.html) VCC, BK, GND可以接在面包板电源上

# 代码

## 显示 Hello RASPBERRY PI
需要安装[RPi.GPIO](https://pypi.python.org/pypi/RPi.GPIO)库

```
#!/usr/bin/python
# -*- coding: utf-8 -*-

import RPi.GPIO as GPIO
import time
import sys

#gpio's :
SCLK = 18
DIN = 15
DC = 13
RST = 11

font =[
0x7E, 0x11, 0x11, 0x11, 0x7E, # A
0x7F, 0x49, 0x49, 0x49, 0x36, # B
0x3E, 0x41, 0x41, 0x41, 0x22, # C
0x7F, 0x41, 0x41, 0x22, 0x1C, # D
0x7F, 0x49, 0x49, 0x49, 0x41, # E
0x7F, 0x09, 0x09, 0x09, 0x01, # F
0x3E, 0x41, 0x49, 0x49, 0x7A, # G
0x7F, 0x08, 0x08, 0x08, 0x7F, # H
0x00, 0x41, 0x7F, 0x41, 0x00, # I
0x20, 0x40, 0x41, 0x3F, 0x01, # J
0x7F, 0x08, 0x14, 0x22, 0x41, # K
0x7F, 0x40, 0x40, 0x40, 0x40, # L
0x7F, 0x02, 0x0C, 0x02, 0x7F, # M
0x7F, 0x04, 0x08, 0x10, 0x7F, # N
0x3E, 0x41, 0x41, 0x41, 0x3E, # O
0x7F, 0x09, 0x09, 0x09, 0x06, # P
0x3E, 0x41, 0x51, 0x21, 0x5E, # Q
0x7F, 0x09, 0x19, 0x29, 0x46, # R
0x46, 0x49, 0x49, 0x49, 0x31, # S
0x01, 0x01, 0x7F, 0x01, 0x01, # T
0x3F, 0x40, 0x40, 0x40, 0x3F, # U
0x1F, 0x20, 0x40, 0x20, 0x1F, # V
0x3F, 0x40, 0x38, 0x40, 0x3F, # W
0x63, 0x14, 0x08, 0x14, 0x63, # X
0x07, 0x08, 0x70, 0x08, 0x07, # Y
0x61, 0x51, 0x49, 0x45, 0x43, # Z
]

def main():
  begin(0xbc) # contrast - may need tweaking for each display
  gotoxy(28,0)
  text("HELLO")
  gotoxy(8,2)
  text("RASPBERRY PI")
  gotoxy(6,2)
  text("Hugo")

def gotoxy(x,y):
  lcd_cmd(x+128)
  lcd_cmd(y+64)

def text(words):
  for i in range(len(words)):
#    print (words[i])
    display_char(words[i])

def display_char(char):
  index=(ord(char)-65)*5
  if ord(char) >=65 and ord(char) <=90:
    for i in range(5):
#      print (index+i)
      lcd_data(font[index+i])
    lcd_data(0) # space inbetween characters
  elif ord(char)==32:
      lcd_data(0)
      lcd_data(0)
      lcd_data(0)
      lcd_data(0)
      lcd_data(0)
      lcd_data(0)

def cls():
  gotoxy(0,0)
  for i in range(84):
    for j in range(6):
      lcd_data(0)

def setup():
  # set pin directions
  GPIO.setmode(GPIO.BOARD)
  GPIO.setup(DIN, GPIO.OUT)
  GPIO.setup(SCLK, GPIO.OUT)
  GPIO.setup(DC, GPIO.OUT)
  GPIO.setup(RST, GPIO.OUT)

def begin(contrast):
  setup()
  # toggle RST low to reset
  GPIO.output(RST, False)
  time.sleep(0.100)
  GPIO.output(RST, True)
  lcd_cmd(0x21) # extended mode
  lcd_cmd(0x14) # bias
  lcd_cmd(contrast) # vop
  lcd_cmd(0x20) # basic mode
  lcd_cmd(0xc) # non-inverted display
  cls()


def SPI(c):
  # data = DIN
  # clock = SCLK
  # MSB first
  # value = c
  for i in xrange(8):
    GPIO.output(DIN, (c & (1 << (7-i))) > 0)
    GPIO.output(SCLK, True)
    GPIO.output(SCLK, False)

def lcd_cmd(c):
#  print ("lcd_cmd sent :",hex(c))
  GPIO.output(DC, False)
  SPI(c)

def lcd_data(c):
#  print ("data sent :",hex(c))
  GPIO.output(DC, True)
  SPI(c)

if __name__ == "__main__":
  main()
```

## 显示树莓派Logo
需要安装wiringPi[https://github.com/WiringPi/WiringPi]和[PCD8544](https://github.com/binerry/RaspberryPi/tree/master/libraries/c/PCD8544)库


```
#include <wiringPi.h>
#include <stdio.h>
#include <stdlib.h>
#include "PCD8544.h"

// pin setup
int _din = 3;
int _sclk = 5;
int _dc = 2;
int _rst = 0;
int _cs = 1;
  
// lcd contrast 
int contrast = 50;
  
int main(int argc, char* argv[])
{
  // print infos
  printf("Raspberry Pi PCD8544 test\n");
  printf("========================================\n");
  
  printf("CLK on Port %i \n", _sclk);
  printf("DIN on Port %i \n", _din);
  printf("DC on Port %i \n", _dc);
  printf("CS on Port %i \n", _cs);
  printf("RST on Port %i \n", _rst);  
  printf("========================================\n");
  
  // check wiringPi setup
  if (wiringPiSetup() == -1)
  {
	printf("wiringPi-Error\n");
    exit(1);
  }
  
  // init and clear lcd
  LCDInit(_sclk, _din, _dc, _cs, _rst, contrast);
  LCDclear();

  // turn all the pixels on (a handy test)
  printf("Test: All pixels on.\n");
  LCDcommand(PCD8544_DISPLAYCONTROL | PCD8544_DISPLAYALLON);
  delay(1000);
  // back to normal
  printf("Test: All pixels off.\n");
  LCDcommand(PCD8544_DISPLAYCONTROL | PCD8544_DISPLAYNORMAL);
  LCDclear();
  
  // display logo
  printf("Test: Display logo.\n");
  LCDshowLogo();
  delay(2000);
  LCDclear();
}
```

# 安装效果
<img src="http://ww1.sinaimg.cn/bmiddle/6bc40342jw1e30zljajskj.jpg"/>

# 参考链接
1. http://binerry.de/post/25787954149/pcd8544-library-for-raspberry-pi