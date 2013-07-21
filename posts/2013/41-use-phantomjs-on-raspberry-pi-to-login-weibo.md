---
date: 2013-07-21
layout: post
title: 在树莓派上使用Phantomjs自动登录微博
description: Use phantomjs on Raspberry Pi to login weibo automatically
categories:
- Blog
tags:
- Javascript

---

{:toc}

使用过新浪开放平台的朋友都知道用户对小应用（用户数较少的）的授权Token很容易[过期]（http://open.weibo.com/wiki/授权机制说明#.E6.8E.88.E6.9D.83.E6.9C.89.E6.95.88.E6.9C.9F），自动续期要求授权过的用户在过期前重新打开授权页。如果你想实现一个自动备份自己微博的App，就不得每天（周）自己去访问授权页（想死的心都有了吧？）。这里介绍一种通过脚本自动登录微博获取最新oAuth token的方法（需要微博登录名和密码），合适自己玩。将脚本部署在树莓派上后，我再也不用每周都去登录一次授权页了，只是收到报警消息后（经常是帐号被冻结了）需要手动处理一下。

# Phantomjs
<img src="http://phantomjs.org/images/phantomjs-logo.png"/>

[Phantomjs](http://phantomjs.org/) 是一个开源的，没有界面可运行在命令行，跨平台，基于WebKit的全功能浏览器，可以用来做网站自动化测试。从源代码[编译](http://phantomjs.org/build.html)比较费时间，可以直接下载[二进制版本](http://phantomjs.org/download.html)，树莓派的版本在[这里](https://github.com/aeberhardo/phantomjs-linux-armv6l)可下载。Phantomjs下载好了后就一个可执行文件，依赖非常少，我很喜欢这种方式。


# 代码

以下代码使用提供的微博用户名和密码登录，获得Token后还会打开微博首页看帐号是否被冻结了。

```
var page = require('webpage').create(),
    system = require('system'),
    fs = require('fs'),
    address;

var weibo_userid = system.args[1]
var weibo_passwd = system.args[2]

var startUrl = "https://api.weibo.com/oauth2/authorize?client_id=<your_app_key>&redirect_uri=<your_return_url>/&response_type=token";

var verify_weibo_freeze = false;

page.onResourceReceived = function (res,network) {
    if (res.stage == "end") {
        // console.log("\t<-" + res.url);
        if (res.url.indexOf("authorize?client_id")>0) {
            startUrl = res.url
        } 
        if (res.url.indexOf("?access_token")>0) {
            var pos1 = res.url.indexOf("access_token=")
            var pos2 = res.url.indexOf("&")
            var access_token = res.url.substring(pos1+"access_token=".length, pos2)
            console.log(weibo_userid + " login OK, access_token is: " + access_token)
            verify_weibo_freeze = true
        }
        if (verify_weibo_freeze && res.url != "http://weibo.com/" && res.url.indexOf("http://weibo.com/")>-1) {
            var pos1 = res.url.indexOf("/",8)
            var pos2 = res.url.indexOf("?")
            var weibo_name = res.url.substring(pos1+1,pos2)
            console.log(weibo_name+" status verified OK")
            phantom.exit();
        }
    }
};

page.onLoadFinished = function() {
    if (verify_weibo_freeze) {
        page.open("http://weibo.com/", function() {
            phantom.exit();
        })
    }
};

page.onConsoleMessage = function(msg) {
    console.log(msg);
};

page.open(startUrl, function(status) {
    if ( status === "success" ) {
        page.includeJs("https://ajax.googleapis.com/ajax/libs/jquery/1.6.1/jquery.min.js", function() {
            var offset = page.evaluate(function(a,b) {
                $("#userId").val(a)
                $("#passwd").val(b)
                if ($('.WB_btn_login').hasClass("formbtn_01")) {
                    // console.log("Found button!")
                    return $('.WB_btn_login').offset()
                }
                return undefined
            }, weibo_userid, weibo_passwd);
            page.sendEvent('click', offset.left + 1, offset.top + 1);
        });
    }
})

```

# 执行方法

`timeout 120 phantomjs weibo_login.js <your_weibo_login_id> <your_weibo_login_password>`

```
<your_weibo_login_id> login OK, access_token is: <your_access_token>
<your_weibo_login_name> status verified OK
```

使用`timeout`要求命令在120秒内完成，防止phantomjs进程遇到问题时一直不退出，实际使用可以用Crontab定时每天执行一次，执行输出可以pipe到另一个脚本，如果不OK则发短信或邮件提示。