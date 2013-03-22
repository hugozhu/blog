---
date: 2013-03-23
layout: post
title: 如何使用Raspberry Pi在1602液晶屏上显示当前时间--电子钟
description: 当然也可以用来显示温度，CPU Load等数据了
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

# 硬件准备

需要以下硬件：

1. [树莓派](http://detail.tmall.com/item.htm?spm=a220m.1000858.1000725.6.nGxekg&id=17337394004&is_b=1&cat_id=2&q=%CA%F7%DD%AE%C5%C9&rn=4004716f9ba818c1d69b5eb7818891b5)
2. [面包板](http://list.tmall.com//search_product.htm?q=%C3%E6%B0%FC%B0%E5&type=p&style=&cat=all)
3. [1602液晶屏](http://s.taobao.com/search?spm=a230r.1.8.2.RvXL2p&q=1602+%D2%BA%BE%A7&commend=all&source=haiwaigou&ssid=s5-e&pid=mm_14507416_2297358_8935934&newpre=null&tab=mall)一块
4. [10K电位器](http://s.taobao.com/search?source=suggest&haiwaifrom=1&q=%B5%E7%CE%BB%C6%F7+10k&initiative_id=staobaoz_20130323&suggest_query=%B5%E7%CE%BB%C6%F7+10k&sb_id=10&suggest=0_2&wq=%B5%E7%CE%BB%C6%F7&tab=mall)
5. [杜邦线](http://list.tmall.com/search_product.htm?q=%B6%C5%B0%EE%CF%DF&user_action=initiative&at_topsearch=1&sort=st&type=p&cat=&style=)
6. [排针](http://s.taobao.com/search?source=haiwaigou&haiwaifrom=1&q=%C5%C5%D5%EB&initiative_id=staobaoz_20130323&tab=mall)
7. [面包板电源](http://list.tmall.com//search_product.htm?q=%C3%E6%B0%FC%B0%E5+%B5%E7%D4%B4&type=p&style=&cat=all)

# 1602 LCD液晶屏

LCD1602液晶屏提供了16列x2行的ASCII字符显示能力，工作电压5V，提供4位数据与8位数据两种工作模式，因为Raspberry Pi的GPIO口数量很有限，所以使用4位数据模式。LCD1602液晶屏模块提供了16个引脚，我们只需接其中的12个即可--请参考[GPIO命名规则](http://hugozhu.myalert.info/2013/03/22/19-raspberry-pi-gpio-port-naming.html)：

1. VSS，接地，RPi PIN 6
2. VDD，接5V电源，PRi PIN 2
3. VO，液晶对比度调节，接电位器中间的引脚
4. RS，寄存器选择，接GPIO 14，RPi PIN 8
5. RW，读写选择，接地，表示写模式，PRi PIN 6
6. EN，使能信号，接GPIO 15，RPi PIN 10
7. D0，数据位0，4位工作模式下不用，不接
8. D1，数据位1，4位工作模式下不用，不接
9. D2，数据位2，4位工作模式下不用，不接
10. D3，数据位3，4位工作模式下不用，不接
11. D4，数据位4，接GPIO 17，RPi PIN 11
12. D5，数据位5，接GPIO 18，RPi PIN 12
13. D6，数据位6，接GPIO 27，RPi PIN 13
14. D7，数据位7，接GPIO 22，RPi PIN 15
15. A，液晶屏背光+，接5V，RPi PIN 2
16. K，液晶屏背光-，接地，RPi PIN 6

# 注意事项
1. 电源VDD最后接上
2. 排针焊接在液晶屏时注意不要虚焊，也可以用万用表测量一下
3. RW脚注意一定要接地
4. 调节电位器可以调节液晶对比度

# 电路图

<img src="http://learn.adafruit.com/system/assets/assets/000/001/729/medium640/pi-char-lcd.gif?1345220594"/>

# 代码

```
#!/usr/bin/python

#
# based on code from lrvick and LiquidCrystal
# lrvic - https://github.com/lrvick/raspi-hd44780/blob/master/hd44780.py
# LiquidCrystal - https://github.com/arduino/Arduino/blob/master/libraries/LiquidCrystal/LiquidCrystal.cpp
#

from time import sleep
from datetime import datetime
from time import sleep

class Adafruit_CharLCD:

    # commands
    LCD_CLEARDISPLAY 		= 0x01
    LCD_RETURNHOME 		    = 0x02
    LCD_ENTRYMODESET 		= 0x04
    LCD_DISPLAYCONTROL 		= 0x08
    LCD_CURSORSHIFT 		= 0x10
    LCD_FUNCTIONSET 		= 0x20
    LCD_SETCGRAMADDR 		= 0x40
    LCD_SETDDRAMADDR 		= 0x80

    # flags for display entry mode
    LCD_ENTRYRIGHT 		= 0x00
    LCD_ENTRYLEFT 		= 0x02
    LCD_ENTRYSHIFTINCREMENT 	= 0x01
    LCD_ENTRYSHIFTDECREMENT 	= 0x00

    # flags for display on/off control
    LCD_DISPLAYON 		= 0x04
    LCD_DISPLAYOFF 		= 0x00
    LCD_CURSORON 		= 0x02
    LCD_CURSOROFF 		= 0x00
    LCD_BLINKON 		= 0x01
    LCD_BLINKOFF 		= 0x00

    # flags for display/cursor shift
    LCD_DISPLAYMOVE 		= 0x08
    LCD_CURSORMOVE 		= 0x00

    # flags for display/cursor shift
    LCD_DISPLAYMOVE 		= 0x08
    LCD_CURSORMOVE 		= 0x00
    LCD_MOVERIGHT 		= 0x04
    LCD_MOVELEFT 		= 0x00

    # flags for function set
    LCD_8BITMODE 		= 0x10
    LCD_4BITMODE 		= 0x00
    LCD_2LINE 			= 0x08
    LCD_1LINE 			= 0x00
    LCD_5x10DOTS 		= 0x04
    LCD_5x8DOTS 		= 0x00



    def __init__(self, pin_rs=8, pin_e=10, pins_db=[11,12,13,15], GPIO = None):
	# Emulate the old behavior of using RPi.GPIO if we haven't been given
	# an explicit GPIO interface to use
	if not GPIO:
	    import RPi.GPIO as GPIO
        GPIO.setwarnings(False)

   	self.GPIO = GPIO
        self.pin_rs = pin_rs
        self.pin_e = pin_e
        self.pins_db = pins_db

        self.GPIO.setmode(GPIO.BOARD)
        self.GPIO.setup(self.pin_e, GPIO.OUT)
        self.GPIO.setup(self.pin_rs, GPIO.OUT)

        for pin in self.pins_db:
            self.GPIO.setup(pin, GPIO.OUT)

	self.write4bits(0x33) # initialization
	self.write4bits(0x32) # initialization
	self.write4bits(0x28) # 2 line 5x7 matrix
	self.write4bits(0x0C) # turn cursor off 0x0E to enable cursor
	self.write4bits(0x06) # shift cursor right

	self.displaycontrol = self.LCD_DISPLAYON | self.LCD_CURSOROFF | self.LCD_BLINKOFF

	self.displayfunction = self.LCD_4BITMODE | self.LCD_1LINE | self.LCD_5x8DOTS
	self.displayfunction |= self.LCD_2LINE

	""" Initialize to default text direction (for romance languages) """
	self.displaymode =  self.LCD_ENTRYLEFT | self.LCD_ENTRYSHIFTDECREMENT
	self.write4bits(self.LCD_ENTRYMODESET | self.displaymode) #  set the entry mode

        self.clear()


    def begin(self, cols, lines):

	if (lines > 1):
		self.numlines = lines
    		self.displayfunction |= self.LCD_2LINE
		self.currline = 0


    def home(self):

	self.write4bits(self.LCD_RETURNHOME) # set cursor position to zero
	self.delayMicroseconds(3000) # this command takes a long time!
	

    def clear(self):

	self.write4bits(self.LCD_CLEARDISPLAY) # command to clear display
	self.delayMicroseconds(3000)	# 3000 microsecond sleep, clearing the display takes a long time


    def setCursor(self, col, row):

	self.row_offsets = [ 0x00, 0x40, 0x14, 0x54 ]

	if ( row > self.numlines ): 
		row = self.numlines - 1 # we count rows starting w/0

	self.write4bits(self.LCD_SETDDRAMADDR | (col + self.row_offsets[row]))


    def noDisplay(self): 
	""" Turn the display off (quickly) """

	self.displaycontrol &= ~self.LCD_DISPLAYON
	self.write4bits(self.LCD_DISPLAYCONTROL | self.displaycontrol)


    def display(self):
	""" Turn the display on (quickly) """

	self.displaycontrol |= self.LCD_DISPLAYON
	self.write4bits(self.LCD_DISPLAYCONTROL | self.displaycontrol)


    def noCursor(self):
	""" Turns the underline cursor on/off """

	self.displaycontrol &= ~self.LCD_CURSORON
	self.write4bits(self.LCD_DISPLAYCONTROL | self.displaycontrol)


    def cursor(self):
	""" Cursor On """

	self.displaycontrol |= self.LCD_CURSORON
	self.write4bits(self.LCD_DISPLAYCONTROL | self.displaycontrol)


    def noBlink(self):
	""" Turn on and off the blinking cursor """

	self.displaycontrol &= ~self.LCD_BLINKON
	self.write4bits(self.LCD_DISPLAYCONTROL | self.displaycontrol)


    def noBlink(self):
	""" Turn on and off the blinking cursor """

	self.displaycontrol &= ~self.LCD_BLINKON
	self.write4bits(self.LCD_DISPLAYCONTROL | self.displaycontrol)


    def DisplayLeft(self):
	""" These commands scroll the display without changing the RAM """

	self.write4bits(self.LCD_CURSORSHIFT | self.LCD_DISPLAYMOVE | self.LCD_MOVELEFT)


    def scrollDisplayRight(self):
	""" These commands scroll the display without changing the RAM """

	self.write4bits(self.LCD_CURSORSHIFT | self.LCD_DISPLAYMOVE | self.LCD_MOVERIGHT);


    def leftToRight(self):
	""" This is for text that flows Left to Right """

	self.displaymode |= self.LCD_ENTRYLEFT
	self.write4bits(self.LCD_ENTRYMODESET | self.displaymode);


    def rightToLeft(self):
	""" This is for text that flows Right to Left """
	self.displaymode &= ~self.LCD_ENTRYLEFT
	self.write4bits(self.LCD_ENTRYMODESET | self.displaymode)


    def autoscroll(self):
	""" This will 'right justify' text from the cursor """

	self.displaymode |= self.LCD_ENTRYSHIFTINCREMENT
	self.write4bits(self.LCD_ENTRYMODESET | self.displaymode)


    def noAutoscroll(self): 
	""" This will 'left justify' text from the cursor """

	self.displaymode &= ~self.LCD_ENTRYSHIFTINCREMENT
	self.write4bits(self.LCD_ENTRYMODESET | self.displaymode)


    def write4bits(self, bits, char_mode=False):
        """ Send command to LCD """

	self.delayMicroseconds(1000) # 1000 microsecond sleep

        bits=bin(bits)[2:].zfill(8)

        self.GPIO.output(self.pin_rs, char_mode)

        for pin in self.pins_db:
            self.GPIO.output(pin, False)

        for i in range(4):
            if bits[i] == "1":
                self.GPIO.output(self.pins_db[::-1][i], True)

	self.pulseEnable()

        for pin in self.pins_db:
            self.GPIO.output(pin, False)

        for i in range(4,8):
            if bits[i] == "1":
                self.GPIO.output(self.pins_db[::-1][i-4], True)

	self.pulseEnable()


    def delayMicroseconds(self, microseconds):
	seconds = microseconds / float(1000000)	# divide microseconds by 1 million for seconds
	sleep(seconds)


    def pulseEnable(self):
	self.GPIO.output(self.pin_e, False)
	self.delayMicroseconds(1)		# 1 microsecond pause - enable pulse must be > 450ns 
	self.GPIO.output(self.pin_e, True)
	self.delayMicroseconds(1)		# 1 microsecond pause - enable pulse must be > 450ns 
	self.GPIO.output(self.pin_e, False)
	self.delayMicroseconds(1)		# commands need > 37us to settle


    def message(self, text):
        """ Send string to LCD. Newline wraps to second line"""

        for char in text:
            if char == '\n':
                self.write4bits(0xC0) # next line
            else:
                self.write4bits(ord(char),True)


if __name__ == '__main__':

    lcd = Adafruit_CharLCD()
    lcd.noBlink()

    while True:
        sleep(1)
        lcd.clear()        
        lcd.message(datetime.now().strftime('  %I : %M : %S \n%a %b %d %Y'))
        
```

# 完成后的效果

<img src="http://ww3.sinaimg.cn/mw690/6bc40342jw1e2xngyc634j.jpg"/>

# 参考链接
1. http://learn.adafruit.com/drive-a-16x2-lcd-directly-with-a-raspberry-pi/wiring