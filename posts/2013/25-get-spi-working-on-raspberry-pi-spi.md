---
date: 2013-04-05
layout: post
title: 在Raspberry Pi上使用硬件SPI
description: Getting SPI working on Raspberry Pi
categories:
- Blog
tags:
- Rasperry Pi

---

# 什么是SPI
SPI (Serial Peripheral Interface)，是一种高速，全双工，同步的通信总线协议，基于SPI的设备需要4根线：

<img src="https://www.evernote.com/shard/s26/sh/61dba789-b17b-4570-b124-03925a97a4bf/4e64405c3b1a486f7aa4b9451e142b32/deep/0/Screenshot%204/5/13%204:27%20PM.jpg
"/>

1. SDO / MOSI - 主设备数据输出，从设备数据输入
2. SDI / MISO - 主设备数据输入，从设备数据输出
3. SCLK / CLK - 时钟信号，由主设备产生
4. CS / SS - 从设备使能信号，由主设备控制

通过CS，主设备可以控制和哪个从设备通信。

## Bit Banging

Bit-banging是一种用软件替代专职硬件的串行通信的技术。软件直接对微处理器的管脚的状态进行设置和采样，其功能涵盖诸如：时钟，电平，同步等所有参数。与此不同的是（传统的串行通信技术中），专职硬件诸如 modem、UART 或者 位移寄存器等一般是用来处理这些参数并且提供一个（缓存）的数据接口，软件在这种情况下同信号处理无关。

bit-banging 具有明显优点诸如：让相同的设备运行不同的协议而只需很小的（甚至不需）硬件的改动。借助很少的额外设备，我们也许可以从数字管脚（数字终端）可以得到视频信号。

bit-banging 也有一些明显的缺点。在软件仿真的过程中消耗的能量比同样功能的专职硬件大。微处理器过忙地从管脚采样和发送采样信号到管脚。在同等微处理器处理能力下，系统常常会有些噪音。

在Rasperry Pi上使用Bit Banging在实际情况下有可能因为操作系统调度造成时钟信号不稳定而使设备收到错误的消息，具体的表现就是Nokia 5110屏在长时间运行过程中出现白屏或花屏现象，如下图：

<img src="http://ww2.sinaimg.cn/bmiddle/6bc40342jw1e3dzvsfxblj.jpg"/>

采用硬件SPI，由Pi的管脚14号Pin（左边倒数第二个）SCLK发出一定频率的时钟信号。经过测试，这种方法产生的时钟信号比Bit Banging软件模拟产生的信号要稳定很多。


<img src="http://ww1.sinaimg.cn/small/6bc40342jw1e3f1s62pnuj.jpg"/>
软件模拟时钟信号波形

<img src="http://ww1.sinaimg.cn/small/6bc40342jw1e3f6d4wsjij.jpg"/>
硬件SPI时钟信号波形


# 测试Pi的硬件SPI

## 确认内核支持
```
root@raspberrypi2 ~/projects/spi_test # ls -la /dev/spi*
crw------- 1 root root 153, 0 Jan  1  1970 /dev/spidev0.0
crw------- 1 root root 153, 1 Jan  1  1970 /dev/spidev0.1
```

## 测试代码 

下载 [spidev_test.c](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/plain/Documentation/spi/spidev_test.c) 或拷贝下面的代码：

```
/*
 * SPI testing utility (using spidev driver)
 *
 * Copyright (c) 2007  MontaVista Software, Inc.
 * Copyright (c) 2007  Anton Vorontsov <avorontsov@ru.mvista.com>
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation; either version 2 of the License.
 *
 * Cross-compile with cross-gcc -I/path/to/cross-kernel/include
 */

#include <stdint.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <getopt.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/types.h>
#include <linux/spi/spidev.h>

#define ARRAY_SIZE(a) (sizeof(a) / sizeof((a)[0]))

static void pabort(const char *s)
{
	perror(s);
	abort();
}

static const char *device = "/dev/spidev1.1";
static uint8_t mode;
static uint8_t bits = 8;
static uint32_t speed = 500000;
static uint16_t delay;

static void transfer(int fd)
{
	int ret;
	uint8_t tx[] = {
		0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
		0x40, 0x00, 0x00, 0x00, 0x00, 0x95,
		0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
		0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
		0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF,
		0xDE, 0xAD, 0xBE, 0xEF, 0xBA, 0xAD,
		0xF0, 0x0D,
	};
	uint8_t rx[ARRAY_SIZE(tx)] = {0, };
	struct spi_ioc_transfer tr = {
		.tx_buf = (unsigned long)tx,
		.rx_buf = (unsigned long)rx,
		.len = ARRAY_SIZE(tx),
		.delay_usecs = delay,
		.speed_hz = speed,
		.bits_per_word = bits,
	};

	ret = ioctl(fd, SPI_IOC_MESSAGE(1), &tr);
	if (ret < 1)
		pabort("can't send spi message");

	for (ret = 0; ret < ARRAY_SIZE(tx); ret++) {
		if (!(ret % 6))
			puts("");
		printf("%.2X ", rx[ret]);
	}
	puts("");
}

static void print_usage(const char *prog)
{
	printf("Usage: %s [-DsbdlHOLC3]\n", prog);
	puts("  -D --device   device to use (default /dev/spidev1.1)\n"
	     "  -s --speed    max speed (Hz)\n"
	     "  -d --delay    delay (usec)\n"
	     "  -b --bpw      bits per word \n"
	     "  -l --loop     loopback\n"
	     "  -H --cpha     clock phase\n"
	     "  -O --cpol     clock polarity\n"
	     "  -L --lsb      least significant bit first\n"
	     "  -C --cs-high  chip select active high\n"
	     "  -3 --3wire    SI/SO signals shared\n");
	exit(1);
}

static void parse_opts(int argc, char *argv[])
{
	while (1) {
		static const struct option lopts[] = {
			{ "device",  1, 0, 'D' },
			{ "speed",   1, 0, 's' },
			{ "delay",   1, 0, 'd' },
			{ "bpw",     1, 0, 'b' },
			{ "loop",    0, 0, 'l' },
			{ "cpha",    0, 0, 'H' },
			{ "cpol",    0, 0, 'O' },
			{ "lsb",     0, 0, 'L' },
			{ "cs-high", 0, 0, 'C' },
			{ "3wire",   0, 0, '3' },
			{ "no-cs",   0, 0, 'N' },
			{ "ready",   0, 0, 'R' },
			{ NULL, 0, 0, 0 },
		};
		int c;

		c = getopt_long(argc, argv, "D:s:d:b:lHOLC3NR", lopts, NULL);

		if (c == -1)
			break;

		switch (c) {
		case 'D':
			device = optarg;
			break;
		case 's':
			speed = atoi(optarg);
			break;
		case 'd':
			delay = atoi(optarg);
			break;
		case 'b':
			bits = atoi(optarg);
			break;
		case 'l':
			mode |= SPI_LOOP;
			break;
		case 'H':
			mode |= SPI_CPHA;
			break;
		case 'O':
			mode |= SPI_CPOL;
			break;
		case 'L':
			mode |= SPI_LSB_FIRST;
			break;
		case 'C':
			mode |= SPI_CS_HIGH;
			break;
		case '3':
			mode |= SPI_3WIRE;
			break;
		case 'N':
			mode |= SPI_NO_CS;
			break;
		case 'R':
			mode |= SPI_READY;
			break;
		default:
			print_usage(argv[0]);
			break;
		}
	}
}

int main(int argc, char *argv[])
{
	int ret = 0;
	int fd;

	parse_opts(argc, argv);

	fd = open(device, O_RDWR);
	if (fd < 0)
		pabort("can't open device");

	/*
	 * spi mode
	 */
	ret = ioctl(fd, SPI_IOC_WR_MODE, &mode);
	if (ret == -1)
		pabort("can't set spi mode");

	ret = ioctl(fd, SPI_IOC_RD_MODE, &mode);
	if (ret == -1)
		pabort("can't get spi mode");

	/*
	 * bits per word
	 */
	ret = ioctl(fd, SPI_IOC_WR_BITS_PER_WORD, &bits);
	if (ret == -1)
		pabort("can't set bits per word");

	ret = ioctl(fd, SPI_IOC_RD_BITS_PER_WORD, &bits);
	if (ret == -1)
		pabort("can't get bits per word");

	/*
	 * max speed hz
	 */
	ret = ioctl(fd, SPI_IOC_WR_MAX_SPEED_HZ, &speed);
	if (ret == -1)
		pabort("can't set max speed hz");

	ret = ioctl(fd, SPI_IOC_RD_MAX_SPEED_HZ, &speed);
	if (ret == -1)
		pabort("can't get max speed hz");

	printf("spi mode: %d\n", mode);
	printf("bits per word: %d\n", bits);
	printf("max speed: %d Hz (%d KHz)\n", speed, speed/1000);

	transfer(fd);

	close(fd);

	return ret;
}
```

用一根杜邦线将Pi的MISO （GPIO 9）和MOSI (GPIO 10)短接，运行上面的代码应该得到如下输出（如果输出不是这样的，那一定是哪里不对了，重启一下看看行不行）

```
root@raspberrypi2 ~/projects/spi_test # ./a.out -D /dev/spidev0.0
spi mode: 0
bits per word: 8
max speed: 500000 Hz (500 KHz)

FF FF FF FF FF FF 
40 00 00 00 00 95 
FF FF FF FF FF FF 
FF FF FF FF FF FF 
FF FF FF FF FF FF 
DE AD BE EF BA AD 
F0 0D 
```

## wiringPiSPI
[wiringPi](https://projects.drogon.net/raspberry-pi/wiringpi/) 软件包提供了SPI使用的帮助类，接口定义如下：

```
int wiringPiSPISetup  (int channel, int speed) ;
int wiringPiSPIDataRW (int channel, unsigned char *data, int len) ;

```

使用则非常简单，下面的代码每秒从Pi的MOSI针脚（第13号，左边倒数第三个）发送1个字节：

```
#include <wiringPi.h>
#include <wiringPiSPI.h>

int main (void)
{

    if (wiringPiSPISetup(0, 5000000) == -1) 
    {
       return -1;
    }
    
    for (;;) 
    {
        uint8_t c = 0x00
        wiringPiSPIDataRW(0, &c, 1);
        delay(1000);
    }
    
}
```

# 实际应用

前面介绍过的Nokia 5110是采用飞利浦PC8544芯片驱动的，就是采用SPI协议的。
采用软件模拟的驱动不是很稳定，改成硬件SPI后就好了，[基于wiringPi的实现](https://github.com/hugozhu/rpi/tree/master/lib/PCD8544), [Go的封装](https://github.com/hugozhu/rpi/tree/master/pcd8544)


# 参考链接
1. http://www.brianhensley.net/2012/07/getting-spi-working-on-raspberry-pi.html
2. 百度百科 - http://baike.baidu.com/view/245026.htm
3. https://zh.wikipedia.org/zh/串行通信
4. https://projects.drogon.net/understanding-spi-on-the-raspberry-pi/