---
date: 2013-04-03
layout: post
title: 在Raspberry Pi上使用Google Channel服务搭建实时应用
description: Setup realtime application with Google Channel Service
categories:
- Blog
tags:
- Golang

---

前面提到了有关个人网站的[实时在线人数](http://hugozhu.myalert.info/2013/03/27/21-realtime-online-user-counter.html)问题，本文要讨论的是如何自己来实现一个这样的统计服务。因为网站也同时部署在Github上，海外用户访问Github镜像网站的访问日志Pi是拿不到的，这怎么办？

# Google Channel Service

[Google Channel Service](https://developers.google.com/appengine/docs/go/channel/overview)允许应用和[GAE (Google App Engine)](http://appengine.google.com/) 保持一个长连接，允许应用实时发送消息给JavaScript客户端，而不用让客户端用效率很低的定时轮询获取新消息。这个服务是允许有多个发布者和多个订阅者，也能创建多个主题来关联发布者和订阅者。

使用这个服务分两步：

1. 客户端请求服务器端（部署在GAE上）获取一个Channel的Token：
<img src="https://developers.google.com/appengine/images/channel_overview01.png"/>

2. 客户端根据Channel Token和服务器建立长连接，并开始接收消息，这时其它的客户端（或服务器端）可以想这个通道发送消息
<img src="https://developers.google.com/appengine/images/channel_overview02.png"/>

# 在线人数统计实现
## 在页面上部署beacon
通常的网站流量统计是依赖部署在页面上的beacon（Javascript或图片标签）来实现的，这样做的好处是可以直接过滤掉一些机器流量，并且可以将日志集中存储在日志收集服务器上，和网站分离开。

于是可以利用GAE实现一个简单的Beacon服务，这里采用Go语言来实现，用Java或Python也是可以的。

```
package counter

import (
	"encoding/base64"
	"fmt"
	"time"
	"net/http"

	"appengine"
	"appengine/channel"
)

var GIF []byte

const (
	TOPIC = "counter"
)

func init() {
	GIF, _ = base64.StdEncoding.DecodeString("R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7")
	
	
	http.HandleFunc("/beacon.gif", handler)
	http.HandleFunc("/new_token", handler_new_token)	
}

func handler(w http.ResponseWriter, r *http.Request) {
	context := appengine.NewContext(r)

	now := time.Now()
	expire := now.AddDate(30, 0, 0)
	zcookie, _ := r.Cookie("z")
	if zcookie == nil {
		zcookie = &http.Cookie{}
		zcookie.Name = "z"
		zcookie.Value = make_hash("<your_salt>", r.RemoteAddr, now.UnixNano())
		zcookie.Expires = expire
		zcookie.Path = "/"
		http.SetCookie(w, zcookie)
	}

	w.Header().Set("Content-type", "image/gif")
	w.Header().Set("Cache-control", "no-cache, must-revalidate")
	w.Header().Set("Expires", "Sat, 26 Jul 1997 05:00:00 GMT")

	fmt.Fprintf(w, "%s", GIF)

	channel.Send(context, TOPIC, zcookie.Value+"\n"+r.RemoteAddr+"\n"+r.Referer()+"\n"+r.UserAgent())
}

func handler_new_token(w http.ResponseWriter, r *http.Request) {
	c := appengine.NewContext(r)
	tok, err := channel.Create(c, TOPIC)	
	callback := r.FormValue("callback")	
	if err != nil {
		http.Error(w, "Couldn't create Channel", http.StatusInternalServerError)
		c.Errorf("channel.Create: %v", err)
		return
	}
	if callback == "" {
		w.Header().Set("Content-type", "text/javascript")
		fmt.Fprintf(w, "%s", tok)
	} else {
		fmt.Fprintf(w, callback+"('%s')", tok)
	}
}
```
代码最后一行是将访问日志实时通过Channel发送出去，该通道有一个指定的主题，这样订阅该主题的客户端都可以收到相应的消息。

```
application: <your_app_name>
version: 1
runtime: go
api_version: go1

handlers:
- url: /(.*).html
  static_files: html/\1.html
  upload: html/(.*\.html)

- url: (/.*)
  script: _go_app
```

将该应用发布到GAE后，可以通过在页面底部放置`<img src="http://<your_app_name>.appspot.com/beacon.gif/>" `标签来将网站的流量引到GAE，并通过Channel发布出去。

## 接收实时消息
比较遗憾的是Google只提供了Javascript库来供浏览器接收实时消息，相应的使用代码如下。

```
<html>
    <body>
        <script type="text/javascript" src="/_ah/channel/jsapi"></script>

        <script>
            function onOpened() {
                alert("onOpened")
            }

            function onMessage(obj) {
                console.log(obj)
                alert("onMessage: " + obj.data)
            }

            function onError(x) {
                alert("onError:"+x)
            }

            function onClose(x) {
                alert("onClose:"+x)
            }

            function init(token) {            
                channel = new goog.appengine.Channel(token);
                socket = channel.open();
                socket.onopen = onOpened;
                socket.onmessage = onMessage;
                socket.onerror = onError;
                socket.onclose = onClose;
            }
        </script>

        <script type="text/javascript" src="/new_token?callback=init"></script>     
       
    </body>
</html>
```
显然Javascript是不能满足统计网站实时流量的需求，我们需要一个服务器端能接收消息的方法，经过对Javascript的逻辑分析，我实现了一个同样能接收消息的[Go语言客户端](https://github.com/hugozhu/gae-rpi-webapp/tree/master/src/google/appengine/channel), 再完善一下后可以发布出来用`go get github.com/hugo/gae-channel`安装。

```
package main

import (
	. "google/appengine/channel"
	"log"
)

func main() {
	log.Println("started")
	stop_chan := make(chan bool)
	channel := NewChannel("http://<your_app_name>.appspot.com/new_token")
	socket := channel.Open()
	socket.OnOpened = func() {
		log.Println("socket opened!")
	}

	socket.OnClose = func() {
		log.Println("socket closed!")
		stop_chan <- true
	}

	socket.OnMessage = func(msg *Element) {
		log.Println(msg.ToString())
	}

	socket.OnError = func(err error) {
		log.Println("error:", err)
	}

	<-stop_chan
}
```

## 实时分析日志
在Pi上利用Channel客户端库，脚本可以实时获取访问日志，可以通过最近15分钟内的日志计算出每分钟独立访客数和PV数，最后可以在Pi的液晶屏幕用柱状图显示出来。


（实现部分待续。。。）



