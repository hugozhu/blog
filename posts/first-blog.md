---
date: 2012-12-22
layout: post
title: 你的第一篇博客
permalink: '/2012/new-born.html'
categories:
- Gor
- Blog
tags:
- Gor
---

感谢你使用Gor编写博客
===================

本文位于 posts/first-blog.md , 你可以任何删掉,修改这个文件
----------------------------------

文件开头是当前文章的元数据

1. date为自动生成, 当然,你可以修改,这是你的自由
2. permalink 可以是固定地址,也可以由gor为你自动生成
3. categories 就是分类, 可以多个
4. tags 同理,多个标签也是很常见的

请确保文件使用UTF8 without BOM编码

你可以通过执行下面的语句来新建一篇博客:
-----------------------------------

	gor post 文章标题

编译你的博客,并预览之
-------------------

	gor compile #编译
	gor http

然后打开你的浏览器,访问 http://127.0.0.1:8080 来预览

你将使用Markdown来编写博客
-------------------------

[Markdown 语法中文版](http://wowubuntu.com/markdown/) 能让你快速入门其语法

相信[MarkdownPad](http://markdownpad.com)或[liteide](http://code.google.com/p/liteide/)会是你的编写博客的好帮手

如果你打算部署到github的pages上
------------------------------

1. 申请github帐户
2. 新建一个库 username.github.com 即你的用户名命名的地址
3. 将compiled目录,作为根路径,提交上去github.com上
4. 稍等几分钟, 你即可通过 http://username.github.com 访问到

附上git教程 [GitBook中文版](http://gitbook.liuhui998.com/)
----------------------------------------------------

一般来说,你只需要几个简单的git命令就足以应付大部分需求(仅示例)

	git clone git://github.com/wendal/wendal.net.git
	git add -A
	git commit -m "..."
	git pull
	git push

用gor编写博客将会是一件很开心的事,如果有任何意见或建议,欢迎到 [gor的官网](http://github.com/wendal/gor) 提交issue
-------------------------------------------------

祝你使用愉快
===========