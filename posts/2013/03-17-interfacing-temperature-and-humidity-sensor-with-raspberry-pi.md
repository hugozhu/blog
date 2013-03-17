---
date: 2013-03-17        
layout: post
title: 如何使用Raspberry Pi测量室内温度和湿度并绘制曲线
description: Interfacing Temperature and Humidity Sensor with Raspberry Pi
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

# 硬件准备

需要以下硬件：

1. 可以工作的[树莓派](http://s.click.taobao.com/t?e=zGU34CA7K%2BPkqB07S4%2FK0CITy7klxxrJ35Nnc0iO6niAHo44Chb01aWIu4ho12MwdcCLV6ff8kJMg0iz0FTGXaJAqMvt94sTe0NIrCAdd8LW)一个
2. 面包板和杜邦线
3. 10K 电位器一个
4. [DHT11温度和湿度传感器]一个

# 传感器电路及原理

## DHT11传感器外观

<img src="http://learn.adafruit.com/system/products/images/000/000/386/medium225/dht11_MED.jpg?1342023044"/>

## 参数

1. 湿度测量范围：20％～90%RH(0－50℃温度补偿）；
2. 温度测量范围：0～+50℃；
3. 湿度测量精度：±5.0%RH
4. 温度测量精度：±2.0℃
5. 响应时间：<5s；


## 电路图

DHT11一共4根引脚，左边第一根接电源5V (Pin 1)，第二根为数据接口，接 Pin 7，第三根不接，第四根接地；在Pin 1和Pin7 直接还需要并联10K的电阻，以保持读数稳定。

<img src="http://learn.adafruit.com/system/assets/assets/000/001/860/original/dht11wiring.gif?1345831788"/>


## 安装好的样子：

<img src="http://ww3.sinaimg.cn/bmiddle/6bc40342jw1e2rx17tlckj.jpg"/>


# 读取温度和湿度的代码

数据读取流图：

<img src="http://1.bp.blogspot.com/-_sMwYSZMGLw/UJpY2RYIA9I/AAAAAAAAAS0/rJ9ZQwZ3IfM/s640/DHT11+timing+diagram.jpg"/>

```
#include <wiringPi.h>  
#include <stdio.h>  
#include <stdlib.h>  
#include <stdint.h>  
#define MAX_TIME 85  
#define DHT11PIN 7  

int dht11_val[5]={0,0,0,0,0};  
int errors=0;

  
void dht11_read_val()  
{  
  uint8_t lststate=HIGH;  
  uint8_t counter=0;  
  uint8_t j=0,i;  
  float farenheit;  
  for(i=0;i<5;i++)  
     dht11_val[i]=0;  
  pinMode(DHT11PIN,OUTPUT);  
  digitalWrite(DHT11PIN,LOW);  
  delay(18);  
  digitalWrite(DHT11PIN,HIGH);  
  delayMicroseconds(40);  
  pinMode(DHT11PIN,INPUT);  
  for(i=0;i<MAX_TIME;i++)  
  {  
    counter=0;  
    while(digitalRead(DHT11PIN)==lststate){  
      counter++;  
      delayMicroseconds(1);  
      if(counter==255)  
        break;  
    }  
    lststate=digitalRead(DHT11PIN);  
    if(counter==255)  
       break;  
    // top 3 transistions are ignored  
    if((i>=4)&&(i%2==0)){  
      dht11_val[j/8]<<=1;  
      if(counter>16)  
        dht11_val[j/8]|=1;  
      j++;  
    }  
  }  
  // verify cheksum and print the verified data  
  if((j>=40)&&(dht11_val[4]==((dht11_val[0]+dht11_val[1]+dht11_val[2]+dht11_val[3])& 0xFF)))  
  {  
    //farenheit=dht11_val[2]*9./5.+32;  
    printf("%d.%d\t%d.%d\n", dht11_val[0],dht11_val[1],dht11_val[2],dht11_val[3]);    
    exit(1);
  }  
  else { 
    errors = errors + 1;
    if (errors > 5) {
      printf("0.0\t0.0");
      exit(2);
    }
  }
}  
  
int main(void)  
{  
  if(wiringPiSetup()==-1)  
    exit(1);  
  while(1)  
  {  
     dht11_read_val();  
     delay(3000);  
  }  
  return 0;  
}  

```

执行`gcc sensor.c -o sensor -lwiringPi` ，运行`sensor`后输出：

```
root@raspberrypi2 /home/hugo/projects/dht11 # ./sensor 
44.0    18.0
```

# 记录曲线图
这里我使用[cosm.com](http://cosm.com/)的服务，注册申请好账号后，可以建立一个datafeed和两个data stream，分别是温度和湿度，相应的Tag ID为1，和2,利用下来的脚本就可以上传数据了

```
#!/bin/bash
####################################################
LOCATION=<填你的程序路径> #home/hugo/projects/dht11
API_KEY='<填你的api_key>'
FEED_ID='<填你的feed_id>'
####################################################
COSM_URL=http://api.cosm.com/v2/feeds/$FEED_ID?timezone=+8

VAL=`$LOCATION/sensor`
t=`echo $VAL|awk '{print $2}'`
h=`echo $VAL|awk '{print $1}'`

STR=`awk 'BEGIN{printf "{\"datastreams\":[ {\"id\":\"1\",\"current_value\":\"%.1f\"}, {\"id\":\"2\",\"current_value\":\"%.1f\"} ] } ",'$t', '$h'}'`

echo $STR
echo $STR > $LOCATION/cosm.json
curl -v --request PUT --header "X-ApiKey: $API_KEY" --data-binary @$LOCATION/cosm.json $COSM_URL
```

然后可以使用如下格式的图片引用把曲线图嵌入任何网页：

```
<img src="https://api.cosm.com/v2/feeds/119331/datastreams/2.png?width=340&height=180&colour=%23f15a24&duration=2days&title=室内湿度&show_axis_labels=false&detailed_grid=true&scale=&timezone=8"/>

```

<img src="https://api.cosm.com/v2/feeds/119331/datastreams/2.png?width=340&height=180&colour=%23f15a24&duration=2days&title=室内湿度&show_axis_labels=false&detailed_grid=true&scale=&timezone=8"/>

<img src="https://api.cosm.com/v2/feeds/119331/datastreams/1.png?width=340&height=180&colour=%23f15a24&duration=2days&title=室内温度&show_axis_labels=false&detailed_grid=true&scale=&timezone=8"/>

# 参考

1. http://www.rpiblog.com/2012/11/interfacing-temperature-and-humidity.html
2. http://learn.adafruit.com/dht-humidity-sensing-on-raspberry-pi-with-gdocs-logging/wiring 