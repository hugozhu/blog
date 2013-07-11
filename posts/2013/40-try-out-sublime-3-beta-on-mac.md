---
date: 2013-07-11
layout: post
title: 在Mac上使用Sublime 3写Go代码
description: Use Sublime 3 on Mac for Go programming
categories:
- Blog
tags:
- Sublime

---

<img src="http://www.sublimetext.com/anim/rename2_packed.png" width="540"/>

[Sublime](http://www.sublimetext.com) 是一个相当好用的文本编辑器，界面简洁，功能强大。最近[Sublime 3 Beta](http://www.sublimetext.com/3) 出来了, 体验了一下，发现启动速度比之前快了很多。

# 下载安装

下载地址： [http://www.sublimetext.com/3](http://www.sublimetext.com/3)

# 安装Package Control

Sublime 支持插件来丰富其功能，[package control](http://wbond.net/sublime_packages/package_control) 本身也是一个插件，可以用来管理其他插件，所以我们要先安装Package Control，Sublime 3需要安装Pacakge Control Alpha.

```
cd "Library/Application Support/Sublime Text 3"
cd Packages/
git clone https://github.com/wbond/sublime_package_control.git "Package Control"
cd "Package Control"
git checkout python3
```

# 安装GoSublime

重启Sublime后

1. 按cmd+shift+p (OS X)或press ctrl+shift+p (Windows, Linux)
2. 在弹出的输入框中输入` PacInstall` 选择 `Package Control: Install Package`
3. 在稍后弹出的输入框内输入"Gosublime"，选择安装

好了，就这样可以开始写Go代码了。


# Tips

1. 我的Sublime配置

	```
	{
		"font_face": "Microsoft YaHei",
		"font_options":
		[
		],
		"font_size": 18.0,
	    "line_padding_top": 0
	}
	```

2. 按快捷键`Shift + Command + L`可以按列编辑

3. 创建`~/bin/subl`软连接

	```
	ln -s /Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl 
	```
	在终端里就可以直接用`subl 文件名|目录` 操作文件和目录了



