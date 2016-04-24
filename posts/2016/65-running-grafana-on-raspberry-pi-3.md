---
date: 2016-04-24
layout: post
title: 用Grafana制作树莓派上的监控大盘
description: Running Grafana on Raspberry Pi 3
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

[Grafana](http://grafana.org)是一个优秀的开源图表系统，支持多种数据源，其中包括
[InfluxDB](https://influxdata.com)和[Prometheus](http://hugozhu.myalert.info/2016/04/17/64-set-up-monitor-system-with-prometheus-on-raspberry.html)

<a href="http://grafana.org/assets/img/features/dashboard_ex.png"><img src="http://grafana.org/assets/img/features/dashboard_ex.png" width="500"/></a>

# 安装

```
export GOPATH=`pwd`
go get github.com/grafana/grafana
cd $GOPATH/src/github.com/grafana/grafana
go run build.go setup              # (only needed once to install godep)
$GOPATH/bin/godep restore          # (will pull down all golang lib dependencies in your current GOPATH)
go run build.go build              # (or 'go build .')
```

编译好后会在bin目录下生成`grafana-server`和`grafana-cli`

第二步需要生成资源文件，但是从源代码编译会遇到`phantomjs-prebuild`依赖包不存在arm版本的问题，可以直接下载windows版本解开里面的public目录到bin/public下

# 启动
在bin/conf下拷贝生成`defaults.ini`文件，执行` ./grafana-server`即可

# 配置
用浏览器打开 `http://<pi:ip>:3000` 用帐号：`admin`，密码：`admin` 登录

首先要配置好`Prometheus`数据源：

<img src="http://ww2.sinaimg.cn/mw690/6bc40342gw1f382aqrlkkj20e80g2myb.jpg"/>

下面是获取网络的上下行流量数据的采集脚本，通过Prometheus Push Gateway提交，可以每5分钟运行一次

```
DATA=`ifconfig eth1 | grep bytes | sed 's/:/ /g' | awk '{print "bytes_rx "$3"\nbytes_tx "$8}'`
cat <<EOF | curl --data-binary @- http://pi3:9091/metrics/job/raspberrypi/instance/test
$DATA
EOF
```
然后在Grafana里可以按下图方式增加一个每小时网络流量的图表，`increase(bytes_rx[1h])`是`Prometheus`所支持的查询表达式。

<a href="http://ww4.sinaimg.cn/mw690/6bc40342gw1f382mb41cvj20lz0ob422.jpg"><img src="http://ww4.sinaimg.cn/mw690/6bc40342gw1f382mb41cvj20lz0ob422.jpg" width="600"/></a>
