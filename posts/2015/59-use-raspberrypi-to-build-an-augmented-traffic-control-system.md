---
date: 2015-03-28
layout: post
title:  使用树莓派搭建低成本，便携，多用户的弱网模拟器：高效测试手机App在弱网下的可用性
description: Use Raspberry Pi to build low cost augmented traffic control system
categories:
- Blog
tags:
- Raspberry Pi


---

{:toc}


# 背景

我们的手机经常会处于弱网情况下，电梯里，高铁上，在地铁站，电影院里。。。如果手机应用不针对弱网做优化，就会出现：白屏页面刷不出来，界面卡死，错误提示一堆，菊花转不停，用户抓狂。。。

移动应用开发团队应该将App在弱网下的可用性作为一个重要的性能指标，在设计和开发阶段考虑在弱网下的体验。

Linux可以使用[`netem`](http://www.linuxfoundation.org/collaborate/workgroups/networking/netem)或`iptables`来实现以下网络模拟：

1. packet delay 网络包延迟
2. packet loss  网络包丢失
3. packet corruption 错误的网络包
4. packet duplication 重复发送网络包
5. packet re-ordering 网络包传输顺序
6. bandwidth 带宽控制

Facebook最近也开源了他们的`augmented traffic control`: [https://github.com/facebook/augmented-traffic-control](https://github.com/facebook/augmented-traffic-control) 主要使用`iptables`和`python`实现，架构合理容易扩展，其控制方法允许多台手机同时使用，并应用不同的网络控制策略；因为提供了RPC接口，在其基础上二次开发也可以较方便的实现自动化弱网测试。

<img src="https://camo.githubusercontent.com/99a779517d18a605046dec36d2beb02166f4e77b/68747470733a2f2f66616365626f6f6b2e6769746875622e696f2f6175676d656e7465642d747261666669632d636f6e74726f6c2f696d616765732f6174635f6f766572766965772e706e67"/>

将树莓派设置成路由器，集成上面提到的软件+二次开发后，我们可以打造出一个低成本，便携，多用户的弱网模拟器。

# 硬件准备

1. 树莓派一只
2. USB无线网卡（芯片使用RealTek RT5370的最方便，RT8188C, RTL8192CU的需要patch hostapd）
3. 8G TF卡
4. 5V/2A充电器
5. Micro USB线
6. 以太网线
7. USB 3G网卡 + SIM卡（可选）
8. 10000mAh 移动电源（可选）

硬件成本300~500元。

# 安装系统

插上后无线网卡后先执行`lsusb`看系统是否已经识别，我们要将无线网卡设置成AP模式，WAN口接外网，让手机通过Wifi接入上网。

```
hugo@raspberrypi2 ~ $ lsusb
...
Bus 001 Device 005: ID 0bda:8178 Realtek Semiconductor Corp. RTL8192CU 802.11n WLAN Adapter
```

修改`/etc/network/interfaces`为以下内容，设置无线LAN的静态IP为`192.168.3.1`

```
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
#iface wlan0 inet manual
#wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
#iface default inet dhcp

iface wlan0 inet static
   address 192.168.3.1
   netmask 255.255.255.0
```

然后安装需要的AP及DNS/DHCP软件：

```
apt-get install hostapd dnsmasq
```

如果你的无线网卡的芯片是RT8188C或RTL8192CU，需要下载一个patch过的[`hostapd`](http://dl.dropbox.com/u/1663660/hostapd/hostapd)替换`/usr/sbin/hostapd` 

## 设置hostapd

1. 修改`/etc/default/hostapd`，最后增加下一行：

```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

2. 新增文件`/etc/hostapd/hostapd.conf`

RT5370芯片的按以下设置：

```
interface=wlan0
driver=nl80211
ssid=Raspberry_AP
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=<YOUR_AP_PASSWD>
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```


RT8188C或RTL8192CU按以下设置：

```
interface=wlan0
driver=rtl871xdrv
bridge=br0
ssid=Raspberry_AP
channel=1
wmm_enabled=0
wpa=2
wpa_passphrase=<YOUR_AP_PASSWD>
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
auth_algs=1
macaddr_acl=0
```

设置好了以后可以执行`service hostapd restart`来验证是否OK。

## 设置dnsmasq
通过无线热点接入进来的手机需要分配IP地址和DNS服务器，这里我们使用`dnsmasq`，配置相当简单：

修改`/etc/dnsmasq.conf`，将DHCP分配范围设置为：`192.168.3.100`到`192.168.3.200`

```
no-resolv
# Interface to bind to
#interface=lo,eth0,wlan0
# Specify starting_range,end_range,lease_time
dhcp-range=192.168.3.100,192.168.3.200,12h
dhcp-option=option:router,192.168.3.1
server=114.114.114.114
server=8.8.8.8
server=8.8.4.4
```

## 设置路由转发

增加下面一行到`/etc/sysctl.conf`

```
net.ipv4.ip_forward=1
```

执行`sysctl -p`使其生效


增加下面的`iptables`规则到`/etc/rc.local`：

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

完成这些设置后，应该可以用手机连上WIFI SSID为`Raspberry AP`的热点，并能够上网。

## 安装弱网模拟系统

```
wget https://bootstrap.pypa.io/get-pip.py
sudo python get-pip.py
sudo pip install atc_thrift atcd django-atc-api django-atc-demo-ui django-atc-profile-storage

django-admin startproject atcui
cd atcui
```

修改`atcui/settings.py`:

```
INSTALLED_APPS = (
    ...
    # Django ATC API
    'rest_framework',
    'atc_api',
    # Django ATC Demo UI
    'bootstrap_themes',
    'django_static_jquery',
    'atc_demo_ui',
    # Django ATC Profile Storage
    'atc_profile_storage',
)
```

修改`atcui/urls.py`:

```
from django.views.generic.base import RedirectView

urlpatterns = patterns('',
    ...
    # Django ATC API
    url(r'^api/v1/', include('atc_api.urls')),
    # Django ATC Demo UI
    url(r'^atc_demo_ui/', include('atc_demo_ui.urls')),
    # Django ATC profile storage
    url(r'^api/v1/profiles/', include('atc_profile_storage.urls')),
    url(r'^$', RedirectView.as_view(url='/atc_demo_ui/', permanent=False)),
)
```

更新Sqlite DB：

```
python manage.py migrate
```

## 自动运行弱网模拟系统

在`/etc/rc.local`中增加下面的命令：

```
/usr/local/bin/atcd --atcd-lan wlan0
cd /home/pi/startproject/atcui
python manage.py runserver 0.0.0.0:80
```

重新启动树莓派后，大约30s后，可以用iPhone连上`Raspberry AP`，然后用Safari打开http://192.168.3.1 你就可以可以看弱网模拟系统的控制面板了。

## 其它设置

关掉网卡的电源管理，防止待机后连不上：

```
hugo@raspberrypi2 ~ $ cat /etc/modprobe.d/8192cu.conf
# Disable power management
options 8192cu rtw_power_mgnt=0
```

# 参考链接
1. http://zh.wikipedia.org/zh-hk/網路流量控制
2. https://github.com/pritambaral/hostapd-rtl871xdrv