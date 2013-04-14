---
date: 2013-04-14
layout: post
title: 使用Go语言在树莓派上编程
description: Programming Raspberry Pi with golang
categories:
- Blog
tags:
- Raspberry Pi

---

[WiringPi](https://projects.drogon.net/raspberry-pi/wiringpi/)是树莓派上比较好的一个开发库，是用C语言写的。使用cgo，我们可以在Go语言里方便的调用WiringPI的函数，于是我包装了一个[WiringPi-Go](https://github.com/hugozhu/rpi)，目前支持wiringPi的基本功能，硬件SPI协议驱动Nokia 5110屏幕，以及中断，未来还会增加PWM和I2C协议的支持。

下面是一个完整的使用例子，结合了之前的两个电路：[链接1](http://hugozhu.myalert.info/2013/04/08/27-interrupts-with-gpio-pins.html)，[链接2](http://hugozhu.myalert.info/2013/04/05/25-get-spi-working-on-raspberry-pi-spi.html)

通过push button可以切换液晶屏显示不同脚本的输出内容。

lcd_switch.go

```
package main

import (
	. "github.com/hugozhu/rpi"
	"github.com/hugozhu/rpi/pcd8544"
	"log"
	"os/exec"
	"time"
)

const (
	DIN        = PIN_MOSI
	SCLK       = PIN_SCLK
	DC         = PIN_GPIO_2
	RST        = PIN_GPIO_0
	CS         = PIN_CE0
	PUSHBUTTON = PIN_GPIO_6
	CONTRAST   = 40 //may need tweak for each Nokia 5110 screen
)

var screen_chan chan int
var TOTAL_MODES = 3

func init() {
	WiringPiSetup()
	pcd8544.LCDInit(SCLK, DIN, DC, CS, RST, CONTRAST)
	screen_chan = make(chan int, 1)
}

func main() {
	//a goroutine to check button push event
	go func() {
		last_time := time.Now().UnixNano() / 1000000
		btn_pushed := 0
		for pin := range WiringPiISR(PUSHBUTTON, INT_EDGE_FALLING) {
			if pin > -1 {
				n := time.Now().UnixNano() / 1000000
				delta := n - last_time
				if delta > 300 { //software debouncing
					log.Println("btn pushed")
					last_time = n
					btn_pushed++
					screen_chan <- btn_pushed % TOTAL_MODES //switch the screen display
				}
			}
		}
	}()

	//a groutine to update display every 5 seconds
	go loop_update_display()

	//set screen 0 to be default display
	screen_chan <- 0

	ticker := time.NewTicker(5 * time.Second)

	for {
		<-ticker.C
		screen_chan <- -1 //refresh current screen every 5 seconds
	}
}

func loop_update_display() {
	current_screen := 0
	for screen := range screen_chan {
		if screen >= 0 {
			if screen != current_screen {
				//btn pushed
				current_screen = screen
				display_loading()
			}
		}
		switch current_screen {
		case 0:
			display_screen0()
		case 1:
			display_screen1()
		case 2:
			display_screen2()
		}
	}
}

func display_loading() {
	pcd8544.LCDclear()
	pcd8544.LCDdrawstring(0, 20, "Loading ...")
	pcd8544.LCDdisplay()
}

func display_screen0() {
	out, err := exec.Command("/bin/screen_0.sh").CombinedOutput()
	if err != nil {
		out = []byte(err.Error())
	}

	pcd8544.LCDclear()
	pcd8544.LCDdrawstring(0, 0, string(out))
	pcd8544.LCDdisplay()
}

func display_screen1() {
	out, err := exec.Command("/bin/screen_1.sh").CombinedOutput()
	if err != nil {
		out = []byte(err.Error())
	}

	pcd8544.LCDclear()
	pcd8544.LCDdrawstring(0, 0, string(out))
	pcd8544.LCDdisplay()
}

func display_screen2() {
	out, err := exec.Command("/bin/screen_2.sh").CombinedOutput()
	if err != nil {
		out = []byte(err.Error())
	}

	pcd8544.LCDclear()
	pcd8544.LCDdrawstring(0, 0, string(out))
	pcd8544.LCDdisplay()
}
```

/bin/screen_2.sh

```
#!/bin/bash

echo "Current IP:" 
ifconfig  | grep "inet " | awk '{print $2}'
```