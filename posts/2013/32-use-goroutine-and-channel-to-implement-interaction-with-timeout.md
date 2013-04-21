---
date: 2013-04-21
layout: post
title: 使用Goroutine和Channel实现按键超时交互
description: Use Goroutine and Channel to implement interaction with timeout
categories:
- Blog
tags:
- Golang

---


# 背景介绍
前面的文章（见[参考链接](#参考链接)）已经介绍了如何使用单触发按键作为树莓派的输入。在实际应用中可以通过按按键循环显示预先设定的脚本输出到显示屏幕，需求如下：

1. 如果按键不被触动，则定时5秒执行脚本获取最新内容显示；
2. 因为不同的脚本获取内容速度会不一样，我们要求如果超过500ms脚本还未返回，需要在屏幕上显示“loading…”这样的过度内容，如果脚本在500ms内返回，则不显示。

使用Goroutine和Channel可以很方便的实现这个需求。

# 代码

```
var screen_chan chan int
var switch_chan = make(chan bool)

func main() {
	//a goroutine： 检查按键是否被按
	go func() {
		last_time := time.Now().UnixNano() / 1000000
		btn_pushed := 0
		total_mode := 3
		for msg := range WiringPiISR(PIN_GPIO_6, INT_EDGE_FALLING) {
			if msg > -1 {
				n := time.Now().UnixNano() / 1000000
				delta := n - last_time
				if delta > 300 { //如果两次按键变化的间隔时间<300ms，是因为接触信号不稳定可以忽略掉
					last_time = n
					btn_pushed++
					screen_chan <- btn_pushed % total_mode
				}
			}
		}
	}()

	//a goroutine： 根据管道消息刷新屏幕
	go loop_update_display()

	//选择确实的屏幕内容脚本编号
	screen_chan <- 0

	//a goroutine: 定时5s向管道发送更新屏幕内容的信号
	ticker := time.NewTicker(5 * time.Second)
	go func() {
		for {
			<-ticker.C
			screen_chan <- -1
		}
	}()
	
	...	
}

func loop_update_display() {
	current_screen := 0
	for msg := range screen_chan {
		switch_screen := false
		if msg >= 0 {
		   //说明是按钮触发的消息，而不是定时器触发的(-1)
			if msg != current_screen {
				//btn pushed
				current_screen = msg
				switch_screen = true
				go func() {
					select {
					case <-time.After(500 * time.Millisecond):
						display_loading()
						<-switch_chan
					case <-switch_chan:
					}
				}()
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
		if switch_screen {
			switch_chan <- true
		}
	}
}
```


```
go func() {
	select {
	case <-time.After(500 * time.Millisecond):
		display_loading(current_screen)
		<-switch_chan
	case <-switch_chan:
	}
}()
```

超时控制的代码就是上面几行了。首先如果是按键触发，主goroutine会创建一个检查超时的goroutine，该goroutine执行`select`语句时会试图从两个管道里获取消息，先获取到消息的管道会继续执行相应分支的代码。

如果`display_screenN`方法在500ms内完成了，`switch_chan`会被主goroutine发送`true`，这时超时检查的goroutine就会直接从`select`语句退出；

如果`display_screenN`方法在500ms内未完成，则`display_loading()`会先执行，执行完后继续等待`switch_chan`的消息，直到`display_screenN`完成。注意`display_xxxxxx`方法内部都使用了互斥锁，有一种边界情况是`display_screenN`先获取了锁，完成显示后`dislay_loading`再执行，这样内容就会一直保持为"loading…"直到下一次定时刷新到来。

如果有其它更好的方法请指教。

# 参考链接

1. http://hugozhu.myalert.info/2013/04/08/27-interrupts-with-gpio-pins.html
2. http://hugozhu.myalert.info/2013/04/14/29-use-wiringpi-go-binding.html