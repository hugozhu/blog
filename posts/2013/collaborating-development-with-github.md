---
date: 2013-02-27
layout: post
title: 使用Github合作开发项目
description: 
categories:
- Blog
tags:
- Github
---

本文大部分内容来自： https://help.github.com/categories/63/articles

Github上合作开发最好的方式是“**Fork + Pull Request**”。比如我最近需要一个静态Blog生成器，市面上有很多Ruby实现的，但我想要一个Go语言的实现，Github上找到了一个 https://github.com/wendal/gor ，测试了一下已有的功能基本能满足了，就用上了。

但实际使用过程中发现了一个问题，如果在URL中有中文，生成的URL如下没有做URL安全编码：

    http://hugozhu.myalert.info/2013/02/27/在Pi和Github上搭建自己的个人博客.html

还有一种情况是如果URL中有空格，如：

    http://hugozhu.myalert.info//2013//02/25/Java properties to enviorment variables.html

浏览器遇到这种URL时，会主动进行编码，但这里有两个问题：

1. 遇到中文时，浏览器是用GBK还是UTF-8还是其它字符集编码后再发送给服务器呢？
2. 遇到空格时，编码成+还是%20呢？
不同浏览器实现可能不一样，在不同操作系统上也可能不一样（可能和用户设置的缺省语言有关），这样有些用户可能会遭遇404错误了，实际上我在服务器的错误日志上的确看到这样的错误

日志：

    2013/02/27 20:41:33 [error] 7791#0: *3285 open() ".../2013/02/25/Java+properties+to+enviorment+variables.html" failed (2: No such file or directory), client: 221.179.193.78, server: hugozhu.myalert.info, request: "GET /2013/02/25/Java+properties+to+enviorment+variables.html HTTP/1.1", host: "hugozhu.myalert.info"

于是我需要动手修改代码：

1. 首先需要做的就是Fork一下原项目到自己的代码仓库： https://github.com/hugozhu/gor
2. 修改好代码并提交到自己的仓库： https://github.com/hugozhu/gor/commit/db2784623d9df4d0652436efdbfbb9caccdc1e1d
3. 在你的代码仓库页面上点Pull Request:
    <img src="https://pbs.twimg.com/media/BEH7V0vCYAAMgcl.jpg:large"/> 
4. 选择好你刚提交好的Commits，然后点发送;
5. 原项目的维护者就会收到这个Pull Request: https://github.com/wendal/gor/pull/14
6. 如果你提交的代码足够好，维护者可以合并到项目主干上；
7. 记住下一次本地修改代码前要先Merge一下原作者新提交的改动;

如下：

    git remote add upstream https://github.com/wendal/gor
    git fetch upstream
    git checkout master
    git merge upstream/master 

到此为止就完成了一次合作开发。

==
我们日常的项目开发中也可以采用这种思路，代码Review也可以增加Pull Request，对项目的迭代速度会有很大帮助。
===
