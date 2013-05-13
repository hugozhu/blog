---
date: 2013-05-13
layout: post
title: 使用8位移位寄存器74HC595扩展树莓派的IO端口
description: Use 74HC595 8bit shift register with Rapsberry Pi
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

树莓派的GPIO接口数目有限，驱动一个步进电机需要占用4个， 一个Nokia 5110液晶也要占4个， 传感器输入至少需要一个，多玩几个外设后接口就不够用了。如果接口可以复用就可以让树莓派驱动更多的外设了，本文讨论如何使用74HC595集成电路芯片来扩展树莓派的I/O接口。


# 芯片介绍
SN74HC595N是德州仪器公司生产的集成电路芯片，是一个8位串行输入变串行输出或并行输出移位寄存器，具有高阻关断，高电平和低电平三态输出。在IO扩充上，可以最多串联15片，也就是高达120个IO扩充。

<img src="https://www.evernote.com/shard/s26/sh/b90034cc-cdf8-426d-a79d-16bbbcbb49b2/0ba3371f3039afa71a17bad845eede7a/deep/0/Screenshot%205/12/13%2010:42%20PM.png"/>

（注意到芯片上的小凹槽了吗，拿芯片的时候以这个为参考物就不会搞反了）

接口的常用命名方式有以下两种：

接口代号(编号)  | 说明     | 接口代号(编号)      | 说明 
---------- | -------- | ------------  | ------------ 
Q7'(9)   |serial data output    | QH'  (9)           | serial data output 
MR (10)	 |Master Reset (Active Low)    | SRCLR  (10)            | Shift register CLeaR
SH_CP (11)          |shift register clock input    | SRCLK  (11)            | Shift Register CLocK input
ST_CP (12)          |storage register clock input    | RCLK  (12)            | storage Register CLocK input
OE (13)          |output enable input (Active Low)| OE  (13)            | Output Enable
DS (14)         |serial data input       | SER (14)             | SERial data input
Qx (15，1-7)         |data output       | Qx (15，1-7)            | data output

# 控制流程
如果要在8个引脚输出01010101

1. 将Pin 14（DS, SER）置为高电平（1）；
2. 将Pin 11 (SH_CP, SRCLK))做高低电平切换，形成一个脉冲信号，这个信号会将数据从移位寄存器C1移动到下一个移位寄存器C2，。。。
3. 接着将Pin 14设为低电平（0），再将Pin 11做1->0的脉冲变化，将数据继续往下移，依次类推直到8位都输入完成；
4. 将Pin 12（ST_CP, RCLK）做1->0的脉冲，将8位数据一次并行输出。

如果要串联多片，由上一片的Pin 9接到下一片的Pin 14即可，这样输入16 bit后，再向Pin 12输入一个1->0的脉冲，16 bit会并行输出。

如果要一次清除所有数据，将Pin 10设为低电平后，再向Pin 12输入一个1->0的脉冲即可；

Pin 13还有高阻关断的第三态输出功能。

下面我们使用一片74HC595来同时控制8个发光二极管的状态，只需要占用树莓派的3个GPIO；否则的话，则需要占用8个。如果需要同时控制16个发光二极管，则可以通过串联两个74HC595来实现。

# 材料
1. [74HC595](http://item.taobao.com/item.htm?spm=a1z0d.1.1000638.45.bTo9PZ&id=3136071980) 因为价格便宜，建议一次买3~10片
2. [面包板](http://list.tmall.com/search_product.htm?q=%C3%E6%B0%FC%B0%E5&type=p&style=&cat=all)
3. [面包板跳线](http://re.taobao.com/eauction?e=WZUyNh6fV9rghojqVNxKsQRl3vWzxAV13G1s8WRbfpWLltG5xFicOSFINJCCZ52POOqhJP6qr9i9NE9AQxtQk3S1BzvCCF6OuLrQdcbQlR6B3ujUJI0OeA%3D%3D&ptype=100011&clk1=ebe7f36b49064c26364fbf2d12bbd3da&upsid=ebe7f36b49064c26364fbf2d12bbd3da) 14根
4. [发光二极管](http://re.taobao.com/eauction?e=NW2xGTCNV2%2FghojqVNxKsRoUGnN%2F%2FLrYlWsbAjdmP56LltG5xFicOSFINJCCZ52POOqhJP6qr9i9NE9AQxtQk3S1BzvCCF6OuLrQdcbQlR6B3ujUJI0OeA%3D%3D&ptype=100011&clk1=231682cf5123eea41a361c2c62bbf6d8&upsid=231682cf5123eea41a361c2c62bbf6d8) 8个
5. [300欧姆电阻](http://re.taobao.com/search?e=NW2xGTCNV2%252FghojqVNxKsRoUGnN%252F%252FLrYlWsbAjdmP56LltG5xFicOSFINJCCZ52POOqhJP6qr9i9NE9AQxtQk3S1BzvCCF6OuLrQdcbQlR6B3ujUJI0OeA%253D%253D&keyword=%B5%E7%D7%E8%B0%FC&type=taoke&refpid=mm_12926928_3484851_11423971&refpos=&unid=0&clk1=231682cf5123eea41a361c2c62bbf6d8&ismall=&catid=&frcatid=) 8个

# 电路图

<img src="https://www.evernote.com/shard/s26/sh/392c6eda-a41a-49ee-b2ec-db61a8dbd94d/353eaa84ddc75fda7b92cdce1a48f293/deep/0/Screenshot%205/12/13%2010:14%20PM.png" width="540"/>


<img src="http://ww2.sinaimg.cn/bmiddle/6bc40342jw1e4luoggxabj20vk0nodm9.jpg"/>

[<img src="http://ww3.sinaimg.cn/bmiddle/6bc40342jw1e4lzscegezj20vk0noqai.jpg"/>](http://photo.weibo.com/1808008002/wbphotos/large/photo_id/3577295178801052?refer=weibofeedv5)

引脚连接

1. 596 Pin 14 -> Raspberry Pi GPIO4
2. 596 Pin 12 -> Raspberry Pi GPIO5
3. 596 Pin 11 -> Raspberry Pi GPIO6

# 代码

```
#include <wiringPi.h>
#include <stdio.h>

int SER   = 4;
int RCLK  = 5;
int SRCLK = 6;

unsigned char LED[8]={0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80};

void SIPO(unsigned char byte);
void pulse(int pin);

void init() {
    pinMode(SER, OUTPUT);
    pinMode(RCLK, OUTPUT);
    pinMode(SRCLK, OUTPUT);

    digitalWrite(SER, 0);
    digitalWrite(SRCLK, 0);
    digitalWrite(RCLK, 0);    
}

void delayMS(int x) {
  usleep(x * 1000);
}

int main (void)
{
    if (-1 == wiringPiSetup()) {
        printf("Setup wiringPi failed!");
        return 1;
    }    

    init();
    int i;
    while(1) {  
      for(i = 0; i < 8; i++)
      {
       SIPO(LED[i]);
       pulse(RCLK);
       delayMS(50);
       printf(" i = %d", i);
      }
      printf("\n");
      delayMS(500); // 500 ms
      
      for(i = 7; i >= 0; i--)
      {
       SIPO(LED[i]);
       pulse(RCLK);
       delayMS(50);
       printf(" i = %d", i);
      }
      delayMS(500); // 500 ms
    }

    usleep(1000);
    digitalWrite(RCLK, 1);
}

void SIPO(unsigned char byte) 
{
    int i;
    for (i=0;i<8;i++) 
    {
        digitalWrite(SER,((byte & (0x80 >> i)) > 0));
        pulse(SRCLK);
    }
}

void pulse(int pin) 
{
    digitalWrite(pin, 1);
    digitalWrite(pin, 0);
}

```
编译执行上面的代码后可以看到LED从左到有一次依次点亮后灭掉，再反向点亮一遍，循环执行，有点像钟摆的效果。

# 参考链接
1. http://baike.baidu.com/view/1309513.htm
2. http://ruten-proteus.blogspot.com/2012/11/io-74hc595-ic.html
3. http://en.wikipedia.org/wiki/Charlieplexing