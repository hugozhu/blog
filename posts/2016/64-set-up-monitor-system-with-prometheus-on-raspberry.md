---
date: 2016-04-17
layout: post
title: 在树莓派3上搭建监控系统
description: Set up monitor system on Raspberry Pi 3 with Prometheus
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

之前用过[tsar](https://github.com/alibaba/tsar)做监控，但没有基于浏览器的图表展示，而且tsar收集数据很频繁，容易写坏SD卡。后来一直用[Xively](https://personal.xively.com) SaaS服务，但因为服务在国外，访问速度一直不尽人意。直到今天早上读到InfoQ的一篇文章才知道了[Prometheus](https://prometheus.io)，看了一下介绍后发现Prometheus的以下特点非常合适在树莓派上部署：

1. 采用Go实现支持，天然支持跨平台，配置相当简单，维护和二次开发的成本小；
2. 采集数据支持Pull和Push模式，可以自定义不同采集点的采样频率，适合轻量型应用降低能耗；
3. 二次计算和查询方式很灵活
4. 自带 `Grafana`数据可视化工具；
5. 可配置的内存+磁盘存储大小，采用的时间序列文件和Level DB做索引效率较高，不会让监控软件本身消耗过多的树莓派计算和存储资源



# 监控系统架构

<img src="http://prometheus.io/assets/architecture.svg">


# 安装
Prometheus采集数据的主要方式是通过HTTP到指定的URL上定时采集，为了支持Push方式收集数据，我们还需要安装一个`Prometheus Pushgateway`作为HTTP服务器来给Prometheus提供数据，你的应用则可以通过命令行或编程接口方式将数据推送到Pushgateway

## 安装Prometheus
到 https://prometheus.io/download/ 选择`armv7`架构后可以直接下载树莓派3可运行的版本，解压后可以直接运行；缺省配置文件将监控Prometheus自身。

## 安装Prometheus Pushgateway
假设你在树莓派上已经安装好了Go

```
git clone https://github.com/prometheus/pushgateway.git
cd pushgateway.git
export GOPATH=`pwd`
go get -d 
go build *.go
```
编译成功后在当前目录下会生成可执行文件：`bindata`

# 配置
Prometheus启动后，缺省用9090 HTTP端口，Prometheus Pushgateway则是9091 HTTP端口, 以下文件配置了每15秒定时抓取Pushgateway上的监控数据。


```
hugo@raspberrypi3:~/prometheus $ cat prometheus.yml
# my global config
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.
  evaluation_interval: 15s # By default, scrape targets every 15 seconds.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'

# Load and evaluate rules in this file every 'evaluation_interval' seconds.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'raspberrypi'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    target_groups:
      - targets: ['localhost:9091']
```      

# 启动

执行下面的命令可以将Prometheus和Push Gateway都启动，数据就可以开始收集了

```
hugo@raspberrypi3:~/prometheus $ cat prometheus.sh
#!/bin/sh
export GOMAXPROCS=2; cd /home/hugo/prometheus; ./prometheus -config.file=prometheus.yml -storage.local.memory-chunks=10240 > /mnt/usb/logs/prometheus.log 2>&1 &
export GOMAXPROCS=2; cd /home/hugo/prometheus; ./bindata > /mnt/usb/logs/prometheus-pg.log 2>&1 &

```

# 使用

有了Push Gateway后，你可以很简单的使用Shell脚本增加监控数据

```
cat <<EOF | curl --data-binary @- http://pi3:9091/metrics/job/raspberrypi
test 2.00
EOF
```

过1分钟后打开 http://pi3:9090/ 就可以看到监控图了。
<img src="http://ww1.sinaimg.cn/mw690/6bc40342gw1f2zs087ncyj20o00lhmzg.jpg"/>

使用这个系统，我们可以很方便的实现室内温度的趋势监控的功能，也可以是抓取股票价格，PM2.5等数据的监控。



