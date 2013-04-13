---
date: 2013-04-13
layout: post
title: 使用tsar记录和监控树莓派CPU温度
description: Use tsar to monitor Raspberry Pi CPU Temperature
categories:
- Blog
tags:
- Raspberry Pi

---

夏天到了，树莓派的CPU温度也开始节节攀升，虽然我们也可以用云服务[Coms](http://hugozhu.myalert.info/2013/03/17/03-17-interfacing-temperature-and-humidity-sensor-with-raspberry-pi.html)来监控，但每5分钟采样一次精度不够高，每分钟采样一次则上传次数又太多了点。最好的方法还是使用[tsar](http://github.com/alibaba/tsar)这样的工具本地高频（如每1分钟）采样，然后再定时将5分钟的均值上传到Coms绘图。

Tsar是淘宝的一个用来收集服务器系统和应用信息的采集报告工具，如收集服务器的系统信息（cpu，mem等），以及应用数据（nginx、swift等），收集到的数据存储在服务器磁盘上，可以随时查询历史信息，也可以将数据发送到nagios报警。Tsar能够比较方便的增加模块，只需要按照tsar的要求编写数据的采集函数和展现函数，就可以把自定义的模块加入到tsar中。

首先按照安装说明，见[https://github.com/alibaba/tsar](https://github.com/alibaba/tsar)将tsar和tsardevel安装好。

首先运行下面的命令生成mod_rpi模块：

```
hugo@raspberrypi2 ~/projects/tsardevel $ tsardevel rpi 
build:make
install:make install
uninstall:make uninstall
hugo@raspberrypi2 ~/projects/tsardevel $ ls rpi
Makefile  mod_rpi.c  mod_rpi.conf
```

然后修改mod_rpi.c，增加读取CPU温度的逻辑：

```
/*
 * (C) 2010-2011 Alibaba Group Holding Limited
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 *
 */

#include "tsar.h"

/*
 * Structure for rpi infomation.
 */
struct stats_rpi {
	unsigned int cpu_temp;
};

#define STATS_TEST_SIZE (sizeof(struct stats_rpi))

static char *rpi_usage = "    --rpi               Rapsberry Pi information (CPU temprature ...)";


static void read_rpi_stats(struct module *mod, char *parameter)
{
	FILE *fp;
	char buf[64];
	memset(buf, 0, sizeof(buf));
	struct stats_rpi st_rpi;
	memset(&st_rpi, 0, sizeof(struct stats_rpi));

	if ((fp = fopen("/sys/class/thermal/thermal_zone0/temp", "r")) == NULL) {
		return;
	}

	int cpu_temp;

	fscanf(fp, "%d", &cpu_temp);

	st_rpi.cpu_temp = cpu_temp;

	int pos = sprintf(buf, "%u",
			/* the store order is not same as read procedure */
			st_rpi.cpu_temp);
	buf[pos] = '\0';
	set_mod_record(mod, buf);
	fclose(fp);
	return;
}

static struct mod_info rpi_info[] = {
	{"  temp", SUMMARY_BIT,  0,  STATS_NULL}
};

static void set_rpi_record(struct module *mod, double st_array[],
		U_64 pre_array[], U_64 cur_array[], int inter)
{
	st_array[0] = cur_array[0]/1000.0;
}

void mod_register(struct module *mod)
{	
	register_mod_fileds(mod, "--rpi", rpi_usage, rpi_info, 1, read_rpi_stats, set_rpi_record);
}

```

最后`make && sudo make install`将mod_rpi自定义tsar模块安装好。

执行`tsar --rpi -l -i 1`可以看到每秒一次输出的实时CPU温度

```
hugo@raspberrypi2 ~/projects/tsardevel/rpi $ tsar --rpi -l -i 1
Time              ---rpi-- 
Time                temp   
13/04/13-12:57:36  47.08   
13/04/13-12:57:37  47.08   
13/04/13-12:57:38  47.08   
13/04/13-12:57:39  46.54   
13/04/13-12:57:40  47.08   
13/04/13-12:57:41  47.08   
13/04/13-12:57:42  47.62   
13/04/13-12:57:43  47.08   
```

执行`tsar`则可以看到间隔为5分钟的历史值

```
Time           ---cpu-- ---mem-- ---tcp-- -----traffic---- mmcblk0- mmcblk0p mmcblk0p --sda---  ---load- ---rpi-- 
Time             util     util   retran    pktin  pktout     util     util     util     util     load1     temp   
13/04/13-12:30   7.77    10.08     5.78     0.00    0.00     8.09     0.00     8.09     1.21      0.28    47.08   
13/04/13-12:35   6.39    10.11     2.53     0.00    0.00     7.70     0.00     7.70     0.23      0.20    47.62   
13/04/13-12:40   5.01    10.11     3.88     0.00    0.00     8.49     0.00     8.49     0.29      0.25    47.62   
13/04/13-12:45   6.24    10.10     3.62     0.00    0.00     6.72     0.00     6.72     0.20      0.09    47.62   
13/04/13-12:50   2.77    10.11     3.99     0.00    0.00     5.20     0.00     5.20     0.16      0.02    47.08   
13/04/13-12:55   2.34    10.09     3.94     0.00    0.00     3.93     0.00     3.93     0.00      0.04    47.62   

MAX             14.68    10.27   100.00     0.00    0.00    18.43     0.00    18.43     1.21      0.56    47.62   
MEAN             2.51     9.08     3.06     0.00    0.00     4.59     0.00     4.59     0.05      0.14     6.15   
MIN              1.67     7.82     0.00     0.00    0.00     3.14     0.00     3.14     0.10      0.08     0.04   
```

tsar很方便吧？ 室温，湿度，网站在线人数等也可以用同样的方法记录。

利用下面的脚本我们还可以把tsar的数据上传到coms:

```
#!/bin/bash
####################################################
# Please customize these values appropriately:
LOCATION=/home/hugo/projects/dht11
API_KEY='<your_api_key>'
FEED_ID='<your_feed_id>'
####################################################
COSM_URL=http://api.cosm.com/v2/feeds/$FEED_ID?timezone=+8

cpu_t=`tsar --rpi | tail -n 5 | awk 'NR==1 {print $2}'`

STR=`awk 'BEGIN{printf "{\"datastreams\":[ {\"id\":\"<your_datapoint_id>\",\"current_value\":\"%.1f\"}]} ",'$cpu_t'}'`

echo $STR
echo $STR > $LOCATION/cosm.json
curl -v --request PUT --header "X-ApiKey: $API_KEY" --data-binary @$LOCATION/cosm.json $COSM_URL

```