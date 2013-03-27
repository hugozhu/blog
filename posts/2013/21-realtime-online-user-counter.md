---
date: 2013-03-27
layout: post
title: 实时网站在线人数接口
description: 把个人网站正在访问的在线人数显示在Nokia 5110液晶屏挺好玩 
categories:
- Blog
tags:

---

感觉把个人网站正在访问的在线人数显示在Nokia 5110液晶屏挺好玩，就稍微研究了一下如何提取实时在线人数。


# 实现方法

## Google Analytics

Google Analytics具有很强大的实时流量分析功能，不过网站主必须登陆到后台才能看，但并没有提供Open API。


## 日志分析
不修改网站通过web服务器的日志分析，用一个脚本统计15分钟内日志的Unique IP可以粗略的获得一个在线人数。
但多个用户可能通过一个IP过来，这种做法肯定不精确。一般我们可以通过在页面上部署Javascript脚本，由Javascript为每一个浏览器产生一个独特的持久化Cookie，用这个Cookie代替IP来统计。但用Raspberry Pi来做这件事情会拖慢网站，于是一种方案是采用免费的Google App Engine来实现。


## CNZZ
CNZZ也提供了15分钟内的在线人数统计功能。分析CNZZ的计数器代码后发现如下方法可以提取到在线人数:

```
curl -s "http://online.cnzz.com/online/online.php?id=[your_cnzz_id]&h=[your_cnzz_server_id].cnzz.com&on=1&s=line" | sed -e 's/.*当前在线\[\([0-9]\).*/\1/g'

```

于是通过脚本提取在线人数并上传到: https://cosm.com/feeds/92372，Cosm有个触发器功能可以当在线人数超过某个值后发Twitter或HTTP Post到指定URL。
