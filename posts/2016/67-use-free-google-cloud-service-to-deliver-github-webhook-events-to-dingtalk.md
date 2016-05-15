---
date: 2016-05-15
layout: post
title: 用免费的Google云服务将Github项目Webhook事件通知到钉钉
description: Use Free Google Cloud Service to deliver Github webhook events to DingTalk
categories:
- Blog
tags:
- DingTalk


{:toc}

本教程使用`Go`语言来实现

![image](http://static.dingtalk.com/media/lALOAQ6nfSvM5Q_229_43.png)

# + 

![image](https://cloud.google.com/_static/b1956bdfe3/images/new-gcp-logo.png)


# 准备开发环境

首先你需要下载Google AppEngine的SDK：`https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Go`

安装好后执行`goapp`确认已安装好

```
4:30:40 Hugo-Mac-mini ~/Projects/hugozhu/godingtalk $ goapp env
GOARCH="amd64"
GOBIN=""
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/hugozhu/Projects/hugozhu/godingtalk"
GORACE=""
GOROOT="/Users/hugozhu/Projects/share/go_appengine/goroot"
GOTOOLDIR="/Users/hugozhu/Projects/share/go_appengine/goroot/pkg/tool/darwin_amd64"
GO15VENDOREXPERIMENT="1"
CC="clang"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fno-common"
CXX="clang++"
CGO_ENABLED="1"

```

## Checkout代码

```
export GOPATH=`pwd`
14:34:49 Hugo-Mac-mini ~/Projects/hugozhu/github_alert $ go get github.com/hugozhu/godingtalk/demo/github/appengine
package appengine: unrecognized import path "appengine" (import path does not begin with hostname)
package appengine/memcache: unrecognized import path "appengine/memcache" (import path does not begin with hostname)
package appengine/urlfetch: unrecognized import path "appengine/urlfetch" (import path does not begin with hostname)
```

## 修改配置

```
cd src/github.com/hugozhu/godingtalk/demo/github/appengine
```

打开`app.yaml`，修改成你的配置

```
application: github-alert-<random_number>
version: 1
runtime: go
api_version: go1

env_variables:
  CORP_ID: '<从 http://oa.dingtalk.com 获取>'
  CORP_SECRET: '<从 http://oa.dingtalk.com 获取>'
  GITHUB_WEBHOOK_SECRET: '<从 http://github.com/ 获取>'
  SENDER_ID: '<从 http://open.dingtalk.com 调用api获取>'
  CHAT_ID: '<从 http://open.dingtalk.com 调用api获取>'

handlers:
- url: /.*
  script: _go_app
```  

## 部署应用

安装`https://cloud.google.com/appengine/docs/go/quickstart`的文档，在Google AppEngine上建立一个`github-alert-<random_number>`项目，使用下面的命令，将看到如下输出

```
14:39:45 Hugo-Mac-mini ~/Projects/hugozhu/github_alert/src/github.com/hugozhu/godingtalk/demo/github/appengine $ goapp deploy
02:43 PM Application: github-alert; version: 1
02:43 PM Host: appengine.google.com
02:43 PM Starting update of app: github-alert, version: 1
02:43 PM Getting current resource limits.
02:43 PM Scanning files on local disk.
02:43 PM Cloning 66 application files.
02:43 PM Uploading 3 files and blobs.
02:43 PM Uploaded 3 files and blobs.
02:43 PM Compilation starting.
02:43 PM Compilation: 65 files left.
02:43 PM Compilation completed.
02:43 PM Starting deployment.
02:44 PM Checking if deployment succeeded.
02:44 PM Deployment successful.
02:44 PM Checking if updated app version is serving.
02:44 PM Completed update of app: github-alert, version: 1
```

部署好后可以用下面的命令验证：

```
curl https://github-alert-<random_number>.appspot.com/github 
Invalid or empty signature
```

## 创建钉钉提醒

回到http://github.com/上找到你的项目设置，将webhook的地址填为`https://github-alert-<random_number>.appspot.com/github`，从此以后你就能在钉钉上接收到项目更新的提醒了。

<img src="http://ww2.sinaimg.cn/mw690/6bc40342gw1f3w2pykihej20k00zkadp.jpg" width="600"/>


