---
date: 2013-04-21
layout: post
title: 树莓派个人网站容灾：DNSPod，利用Google App Engine和Github
description: Use Google App Engine to failover your blog between Raspberry Pi and Github
categories:
- Blog
tags:
- Golang
- Google App Engine

---


# 背景介绍
[把网站托管在树莓派上](http://hugozhu.myalert.info/2013/02/27/在Pi和Github上搭建自己的个人博客.html)后如果家里停电或是宽带故障，会造成网站中断。本文提供一个免费的解决方案（前提是你需要有自己的一个域名，并由DNSPod解析）

# DNSPod

首先需要在DNSPod里设置好需要failover的域名CNAME：比如`hugozhu.myalert.info`
<img src="https://www.evernote.com/shard/s26/sh/70d9eb43-ff76-4d7f-b6a3-34411eca53cd/a89cccd32eeccae3b6ca3627693f2c9a/res/a8eb9052-9bfa-4b64-9e71-150d0c6b13c9/skitch.png?resizeSmall&width=832"/>
其中｀默认｀指向`pi.myalert.info`,这是一个域名的A Record，会由运行在树莓派上的[脚本](http://hugozhu.myalert.info/2013/02/26/dynamic-dns-script.html)来更新动态IP，｀国外｀则指向github。当停电时我们需要自动把｀默认｀这条纪录修改成github。

使用下面命令获得相应CNAME的domain_id：
```
curl curl -k https://dnsapi.cn/Domain.List -d "login_email=xxx&login_password=xxx" 
```

使用下面命令获得相应CNAME的record_id：
```
curl -k https://dnsapi.cn/Record.List -d "login_email=xxx&login_password=xxx&domain_id=xxx"
```

# Google App Engine

## 切换DNS脚本

```
package dnspod

import (
	"io/ioutil"
	"net/http"
	"net/url"
	"strings"
)

const (
	login_email    = "<your_login_email>"
	login_password = "<your_login_password>"
	format         = "json"
	domain_id      = "<domain_id>"
	record_id      = "<record_id>"
	sub_domain     = "<your_subdomain>"
	record_type    = "CNAME"
	record_line    = "默认"
	ttl            = "600"
)

func Update(client *http.Client, cname string) string {
	body := url.Values{
		"login_email":    {login_email},
		"login_password": {login_password},
		"format":         {format},
		"domain_id":      {domain_id},
		"record_id":      {record_id},
		"sub_domain":     {sub_domain},
		"record_type":    {record_type},
		"record_line":    {record_line},
		"value":          {cname},
		"ttl":            {ttl},
	}
	req, err := http.NewRequest("POST", "https://dnsapi.cn/Record.Modify", strings.NewReader(body.Encode()))
	req.Header.Set("Accept", "text/json")
	req.Header.Set("Content-type", "application/x-www-form-urlencoded")
	resp, err := client.Do(req)
	if err != nil {
		return err.Error()
	}
	defer resp.Body.Close()
	bytes, _ := ioutil.ReadAll(resp.Body)
	return string(bytes)
}
```

## 检测接口
部署一个web应用到Google App Engine上，该应用接受树莓派上的一个URL（注意这里不应该用需failver的域名），并请求该域名以看返回是否正常。也可以使用监控宝来监控，但只有付费专业版才支持出错后回调URL。

```
func ping(w http.ResponseWriter, r *http.Request) {
	url := r.FormValue("url")
	c := appengine.NewContext(r)
	client := urlfetch.Client(c)
	ok, body := url_ok(client, url)

	item, _ := memcache.Get(c, "site_fails")

	if ok {
		if item != nil {
			//switch back to pi
			dnspod.Update(client, "pi.myalert.info.")
			memcache.Delete(c, "site_fails")
		}
	} else {
		if item != nil {
			//previously failed, switch to github
			dnspod.Update(client, "hugozhu.github.com.")
			value, _ := strconv.Atoi(string(item.Value))
			value++
			item.Value = []byte(strconv.Itoa(value))
			memcache.Set(c, item)
		} else {
			//first time failed
			item = &memcache.Item{
				Key:   "site_fails",
				Value: []byte("0"),
			}
			memcache.Set(c, item)
		}
	}

	w.Header().Set("Content-type", "text/plain")
	fmt.Fprintf(w, "%s", body)
}

func url_ok(client *http.Client, url string) (bool, string) {
	resp, err := client.Get(url)
	ok := false
	body := ""
	if err != nil {
		body = err.Error()
	} else {
		bytes, _ := ioutil.ReadAll(resp.Body)
		body = string(bytes)
		resp.Body.Close()
		ok = resp.StatusCode == 200
	}
	return ok, body
}
```

## 定时任务

在gae项目里增加一个文件：[cron.yaml](https://github.com/hugozhu/gae-rpi-webapp/blob/master/cron.yaml)，内容如下，这样gae会自动每隔15分钟检查一次，如果连续发现两次不可用，就会切换到Github，恢复后则会切换回来。

```
cron:
- description: check pi if it's still alive
  url: /ping
  schedule: every 15 minutes
```



# 参考链接

1. http://hugozhu.myalert.info/2013/02/27/在Pi和Github上搭建自己的个人博客.html
2. https://github.com/hugozhu/gae-rpi-webapp 