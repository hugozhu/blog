---
date: 2013-03-16
layout: post
title: 如何使用Raspberry Pi控制步进电机旋转高清摄像头并拍照
description: 后续利用OpenCV还可以合成多张照片成全景图，或者多角度连续视频录制
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

# 硬件准备

需要以下硬件：

1. 可以工作的[树莓派](http://s.click.taobao.com/t?e=zGU34CA7K%2BPkqB07S4%2FK0CITy7klxxrJ35Nnc0iO6niAHo44Chb01aWIu4ho12MwdcCLV6ff8kJMg0iz0FTGXaJAqMvt94sTe0NIrCAdd8LW)一个
2. [母对母1P杜邦线](http://s.click.taobao.com/t?e=zGU34CA7K%2BPkqB07S4%2FK0CITy7klxxrJ35Nnc0iO6niAHoWKV0kwS8Wy16Cg6qBM%2BZAOnJCqNG%2BPJAy9U15g8TwJiq5U3GGcJ8eTvC7%2F6APShw%3D%3D)6根
3. [DC 5V4相28YBJ-48步进电机](http://s.click.taobao.com/t?e=zGU34CA7K%2BPkqB07S4%2FK0CFcRfH0GoT805sipKjxjSLt%2BsmFEpvY8zQ4WXjoTHsLWTKD4gnL0sndE3qVPXd4UC6NUEZOQAryYUWhO7gt10i%2FUQ%3D%3D)一个
4. [UL2003芯片步进电机驱动板](http://s.click.taobao.com/t?e=zGU34CA7K%2BPkqB07S4%2FK0CFcRfH0GoT805sipKjxjSLuXQmw2TUIlWTNRCvS2wo483pjZyXspPuvkTH5pg4vQUqrztOAoNz2Gfp8MmwKPg%2FbQb8%3D)一块


# 安装
按下图将步进电机接到驱动板上，也就是白色的接口

<img src="http://img04.taobaocdn.com/bao/uploaded/i4/T10lS4XnXfXXaZhKrX_115008.jpg_310x310.jpg"/>

<img src="http://img02.taobaocdn.com/imgextra/i2/49873130/T2534HXoBXXXXXXXXX_!!49873130.jpg" width="600"/>

## 步进电机电源
步进电机需要5V电压驱动，而树莓派的[GPIO接口](http://elinux.org/RPi_Low-level_peripherals)中已有5V输出，将图中的Pin 2（最右上角那个）5V，接到驱动板的5V正极，Pin 6接到5V负级，电源部分则搞定。

<img src="http://trouch.com/wp-content/uploads/2012/08/webiopi-chrome.png" width="600"/>

## 步进电机驱动线路
驱动板上有IN1, IN2, IN3, IN4四个接口，根据资料得知这四个接口依次设置为低电平就可以驱动，我们分别用杜邦线将GPIO 17（Pin 11），GPIO 18（Pin 12）, GPIO 21（Pin 13）, GPIO 22（Pin 15）和IN1，IN2，IN3，IN4一一相连。 **注意不同的GPIO驱动对端口的编号不一定一样。**

驱动原理：（每次将四个GPIO端口按下表依次设置好电平后，可以sleep几十毫秒来控制转速）

**序列**    | **GPIO 17** | **GPIO 18** | **GPIO 21**|**GPIO 22**  
------------ | ------------- | ------------ | ------------
0            | **LOW**       | HIGH      | HIGH | HIGH
1            | HIGH          | **LOW**   | HIGH | HIGH
2            | HIGH          | HIGH      | **LOW**  | HIGH
3            | HIGH          | HIGH      | HIGH | **LOW**
4            | **LOW**       | HIGH      | HIGH | HIGH
5|…

## 安装摄像头

本来是希望用3D打印机来制作齿轮和支架来完成这部分工作的，但因为打印机还没到货，所以先用乐高积木来做了, 刚好乐高积木可以插在步进电机中轴上，而且很牢靠，还不用密封带了。

摄像头如下图用两根导线固定在乐高积木上：

<img src="http://ww1.sinaimg.cn/bmiddle/6bc40342jw1e2rqxrtrd2j.jpg"/>

然后用各种积木搭个底座把电机固定起来，并留两个洞口可以将驱动线和摄像头的USB线穿出，这样表面上比较整齐，USB线也不会因为牵扯影响转动。

<img src="http://ww4.sinaimg.cn/bmiddle/6bc40342jw1e2rqt5b23rj.jpg"/>

[点击看大图](http://photo.weibo.com/1808008002/wbphotos/large/photo_id/3556528869320348?refer=weibofeedv5)


# 驱动示例代码

这里使用的是Python GPIO库，注意这里的端口命名是按树莓派的叫法（Pin 11, 12, 13, 15）

```
root@raspberrypi2 ~/projects/step_motor # cat motor.py
import RPi.GPIO as GPIO
import time
import sys
from array import *

GPIO.setwarnings(False) 
GPIO.setmode(GPIO.BOARD)

steps    = int(sys.argv[1]);
clockwise = int(sys.argv[2]);

arr = [0,1,2,3];
if clockwise!=1:
    arr = [3,2,1,0];

ports = [11,12,13,15]

for p in ports:
    GPIO.setup(p,GPIO.OUT)

for x in range(0,steps):
    for j in arr:
        time.sleep(0.01)
        for i in range(0,4):
            if i == j:            
                GPIO.output(ports[i],True)
            else:
                GPIO.output(ports[i],False)
```                

执行`python motor.py  90 0` 可以顺时针转动大约80度。

执行`python motor.py  90 1` 则可逆时针转动大约80度。

## 转动效果视频

<embed src="http://player.youku.com/player.php/sid/XNTI3MzU1MjIw/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

# 连续转动拍摄代码实现

这次使用[webiopi](http://code.google.com/p/webiopi/)把控制程序转换成REST API，这样方便网页调用。

```
root@raspberrypi2 ~/projects/gpio_server # cat webiopi_custom.py
# Imports
import webiopi
import time

# Retrieve GPIO lib
GPIO = webiopi.GPIO

# -------------------------------------------------- #
# Macro definition part                              #
# -------------------------------------------------- #

# A custom macro which prints out the arg received and return OK
def myMacroWithArgs(arg1, arg2, arg3):
    print("myMacroWithArgs(%s, %s, %s)" % (arg1, arg2, arg3))
    return "OK"

# A custom macro without args which return nothing
def myMacroWithoutArgs():
    print("myMacroWithoutArgs()")

# Example loop which toggle GPIO 7 each 5 seconds
def loop():
    time.sleep(5)        


def turnLed(port_str, ms):
    port = int(port_str)
    GPIO.setFunction(port,GPIO.OUT)    
    GPIO.output(port,GPIO.LOW)
    time.sleep(float(ms)/1000)
    GPIO.output(port,GPIO.HIGH)

def turnWebcam(steps_str, clockwise_str):
    steps = int(steps_str);
    clockwise = int(clockwise_str);
    arr = [0,1,2,3];
    if clockwise!=1:
        arr = [3,2,1,0];

    ports = [17,18,27,22]

    for p in ports:
        GPIO.setFunction(p,GPIO.OUT)

    for x in range(0,steps):
        for j in arr:
            time.sleep(0.01)
            for i in range(0,4):
                if i == j:            
                    GPIO.output(ports[i],GPIO.LOW)
                else:
                    GPIO.output(ports[i],GPIO.HIGH)

# -------------------------------------------------- #
# Initialization part                                #
# -------------------------------------------------- #

# Setup GPIOs

# -------------------------------------------------- #
# Main server part                                   #
# -------------------------------------------------- #

# Instantiate the server on the port 8000, it starts immediately in its own thread
server = webiopi.Server(port=8001, login="pi", password="pi")
# or     webiopi.Server(port=8000, passwdfile="/etc/webiopi/passwd")

# Register the macros so you can call it with Javascript and/or REST API
server.addMacro(turnWebcam)
server.addMacro(turnLed)


# -------------------------------------------------- #
# Loop execution part                                #
# -------------------------------------------------- #

# Run our loop until CTRL-C is pressed or SIGTERM received
webiopi.runLoop()

# If no specific loop is needed and defined above, just use 
# webiopi.runLoop()
# here instead

# -------------------------------------------------- #
# Termination part                                   #
# -------------------------------------------------- #

# Cleanly stop the server
server.stop()
```

执行`python webiopi_custom.py` 后启动GPIO REST API服务器

转动命令是：`curl --data "" "http://pi:pi@raspberrypi2:8001/macros/turnWebcam/90,0`

拍照命令是：`/usr/bin/fswebcam -v -r 640x480 --no-banner /var/www/fswebcam/foo.jpg`

于是我们可以用以下方法来实现连续拍摄：

1. 执行拍照命令, 生成right.jpg
2. 顺时转80度
3. 执行拍照命令, 生成middle.jpg
4. 顺时转80度
5. 执行拍照命令, 生成left.jpg
6. 逆时针转160度归位

将命令通过网页执行后，就可以在外面看房间里的情况了，今天出去外面采草莓在iPhone上试了一下，结果符合预期。

# 拍摄图片效果

在手机上看到的页面：点Reload会重新连拍三张
<img src="http://ww3.sinaimg.cn/large/6bc40342jw1e2r069hyhsj.jpg"/>


总共花了不到2小时就可以搞定这个了，还是非常好玩的~ 后面还可以用OpenCV库来合成照片到真正的全景图







