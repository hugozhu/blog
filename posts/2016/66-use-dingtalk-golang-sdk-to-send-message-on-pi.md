---
date: 2016-05-02
layout: post
title: 从树莓派上发消息到手机或桌面钉钉
description: Use DingTalk golang sdk to send message to your devices on Raspberry Pi
categories:
- Blog
tags:
- Raspberry Pi

---

{:toc}

![image](http://static.dingtalk.com/media/lALOAQ6nfSvM5Q_229_43.png)

钉钉是阿里巴巴专为中小企业和团队打造的沟通、协同的多端平台，钉钉开放平台旨在为企业提供更为丰富的办公协同解决方案。通过钉钉开放平台，企业或第三方合作伙伴可以帮助企业快速、低成本的实现高质量的移动微应用，实现生产、管理、协作、运营的移动化。

访问钉钉开放平台的文档，<a href="http://open.dingtalk.com">请戳此</a>

下面将介绍如何使用钉钉开放平台SDK在树莓派上发送消息到手机和桌面钉钉。

# 准备工作
首先，你需要在钉钉上创建一个组织，点<a href="https://oa.dingtalk.com/register.html?spm=0.0.0.0.gAZ1E9">这里开始</a>

注册好后创建微应用

<img src="http://ww4.sinaimg.cn/mw690/6bc40342gw1f3h1bdwhinj20kj0dqwhk.jpg"/>

获取微应用的 `agentid`

<img src="http://ww1.sinaimg.cn/mw690/6bc40342gw1f3h1bdr75ej20fm08aaaj.jpg"/>

获取 `corpid`和`corpsecret`，非常重要

<img src="http://ww3.sinaimg.cn/mw690/6bc40342gw1f3h1bdu2dfj20kl0cngmy.jpg"/>

# 下载钉钉SDK
这里我们使用Go语言版的钉钉开放平台SDK，这样可以直接在树莓派上编译运行

```
export GOPATH=`pwd`
go get github.com/hugozhu/godingtalk
```

# 企业应用消息发送代码

下面的代码使用钉钉开放平台的企业应用消息接口来发送消息

```
package main

import (
	"github.com/hugozhu/godingtalk"
	"log"
	"os"
)

func main() {
	c := godingtalk.NewDingTalkClient(os.Getenv("corpid"), os.Getenv("corpsecret"))
	c.RefreshAccessToken()
	err := c.SendAppMessage(os.Args[1], os.Args[2], os.Args[3])
	if err != nil {
		log.Println(err)
	}
}
```

将上面的代码保存在`src/push.go`里，执行`go build src/push.go`生成可执行文件`push`，并复制到`~/bin`目录下


# 消息发送脚本
在准备工作中获取到的`corpid`,`corpsecret` 和 `agentid`这里就有用了

创建一个`push.sh`文件，内容如下：

```
#!/bin/sh

export corpid=<corpid>
export corpsecret=<corpsecret>
timeout 10 ~/bin/push <agentid> "@all" "$1"
```

# 使用消息发送脚本

```
push.sh "树莓派发来的钉钉消息"
```

通过这个脚本就可以在树莓派上发消息到钉钉上了

<img src="http://ww1.sinaimg.cn/mw690/6bc40342gw1f3h1snr9o5j20ku112ae7.jpg"/>

# 参考链接
1. [钉钉开发平台](http://open.dingtalk.com/)

