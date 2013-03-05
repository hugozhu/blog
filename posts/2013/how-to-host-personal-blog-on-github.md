---
date: 2013-02-27
layout: post
title: 在Pi和Github上搭建自己的个人博客
permalink: '/2013/02/27/%E5%9C%A8Pi%E5%92%8CGithub%E4%B8%8A%E6%90%AD%E5%BB%BA%E8%87%AA%E5%B7%B1%E7%9A%84%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2.html'
description: How to host your personal blog on raspberry pi和github
categories:
- Blog
tags:
- Github
---


方法如下：
=======

本站同时托管在家里的[Raspberry Pi](http://www.raspberrypi.org)和[Github Pages](http://pages.github.com/)上，并同步保持更新，海外用户会访问Github，国内用户则会访问Pi，不同线路解析域名**hugozhu.myalert.info**到不同的服务器是通过[DnsPod](http://dnspod.cn)的服务实现的，这么好的服务还是免费的，这里推荐一下。

因为Github Pages只能支持静态网页，你需要一个能生成静态网页的博客生成引擎。我使用的是[gor](http://github.com/wendal/gor) ， 也可以使用[ruhoh](http://ruhoh.com)，[Google一下还有很多](https://www.google.com/search?q=static+blog+generator&hl=en&newwindow=1&client=safari&rls=en&biw=1238&bih=868&ei=msAuUY-vDMKO2AWQ7IHoBQ&sqi=2&start=10&sa=N)。

静态页面博客的好处：
---------------
1. 性能是最好的，很合适用Raspberry Pi来做服务器，节省资源；
2. 文章可以用Markdown格式来编写，采用Github来做版本控制，我的Blog仓库在 http://github.com/hugozhu/blog ，数据安全很好，误删除也不担心了；
3. 很容易找到托管环境，方便迁移；
4. 用Gor在Pi上生成速度很快；再用Nginx提供Web服务，可以直接在Pi上写Blog；
5. 大繁至简

Github设置
--------- 
1. 在你的仓库里增加一个your_github_id.github.com，比如我的github ID是hugozhu，相应的仓库名就是[hugozhu.github.com](https://github.com/hugozhu/hugozhu.github.com)，这个仓库也就是网站的根目录了，在这里放生成好的静态文件
2. 如果你需要用自己的域名，而不是Github提供的，可以在根目录下增加一个[CNAME](https://github.com/hugozhu/hugozhu.github.com/blob/master/CNAME)文件,文件内容则是你的域名，在DnsPod上需要建一个CNAME记录，将你的域名指向your_github_id.github.com. 也就是github原来分配给你的，完成这个设置后，访问your_github_id.github.com会跳转到你的域名；
3. 每次更新后，Github会在10分钟内生效。

更新博客
--------
1. Gor的使用详细说明可见 https://github.com/wendal/gor
2. 我的整个网站的内容也通过Github开源了: https://github.com/hugozhu/blog

以我的网站为例：

    git clone https://github.com/hugozhu/blog
        Cloning into 'blog'...
        remote: Counting objects: 190, done.
        remote: Compressing objects: 100% (146/146), done.
        remote: Total 190 (delta 81), reused 132 (delta 23)
        Receiving objects: 100% (190/190), 155.48 KiB | 171 KiB/s, done.
        Resolving deltas: 100% (81/81), done.
    cd blog
    gor compile
        2013/02/27 13:17:19 gor.go:21: gor ver 2.1
        2013/02/27 13:17:19 payload.go:572: Load Layout : default
        2013/02/27 13:17:19 payload.go:572: Load Layout : page
        2013/02/27 13:17:19 payload.go:572: Load Layout : post
        2013/02/27 13:17:19 config.go:61: Look lile a Json, try it
        2013/02/27 13:17:19 config.go:64: It is Json Map
        2013/02/27 13:17:19 widgets.go:111: Load widget from  widgets/analytics/config.yml
        2013/02/27 13:17:19 widgets.go:111: Load widget from  widgets/comments/config.yml
        2013/02/27 13:17:19 widgets.go:111: Load widget from  widgets/google_prettify/config.yml
        2013/02/27 13:17:19 compile.go:125: Done
    cd compiled
    git init
    git add -A 
    git commit -m "update website" .
    git remote add origin hugozhu@github.com:hugozhu/hugozhu.github.com
    git push -u origin master

最后等待10分钟，再打开 http://hugozhu.github.com 就好了。。。       

Raspberry Pi设置
----------------
安装nginx

    sudo apt-get install nginx
修改文件：/etc/nginx/sites-enabled/default，增加下面内容

    server {
        server_name hugozhu.myalert.info;

        root /home/pi/blog/compiled;
        
        location / {
            ssi on;
        }     
    }
重新启动nginx，这样在Pi上也有一个你的个人博客了，方便自己访问，这里有个小小的技巧是可以通过server side include给静态页面增加动态内容，上面的配置在首页上打开了此功能，这样我可以在页底加上如下代码来显示访问者的IP

    <!--# echo var="remote_addr" default="no" -->
最后重启Nginx生效

    sudo /etc/init.d/nginx restart

TODO:
===
1. 实现一个简单的Web界面，可以通个Web界面来保存Blog，并重现编译和更新到Github；
> 已部分实现：在Github的博客仓库里可以直接创建或修改文件，用Go写了一个HTTP接口，curl一下后可更新，
2. 微博到博客的快速发布；