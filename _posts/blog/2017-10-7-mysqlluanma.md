---
layout: post
title:  Ubuntu安装Navicat for MySQL界面乱码问题
categories: Database
description: 
keywords: 
---


之前使用Mysql都是用的命令行，最近发现太不方便，于是打算装一个图形化客户端。选择了Navicat for MySQL，操作系统是Ubuntu。

解压压缩包后，运行start_navicat文件，结果发现，界面的中文变成了乱码。

以下是解决方法：

用vim或者gedit打开start_navicat文件，找到以下内容：

	export LANG="en_US.UTF-8" 

将这句话改为：

 	export LANG="zh_CN.UTF-8"

问题基本可以解决。

如果以上方法还没解决的话，终端输入locale查看一下，我的是以下内容：

	LANG=zh_CN.UTF-8
	LANGUAGE=zh_CN:
	LC_CTYPE="zh_CN.UTF-8"
	LC_NUMERIC="zh_CN.UTF-8"
	LC_TIME="zh_CN.UTF-8"
	LC_COLLATE="zh_CN.UTF-8"
	LC_MONETARY="zh_CN.UTF-8"
	LC_MESSAGES="zh_CN.UTF-8"
	LC_PAPER="zh_CN.UTF-8"
	LC_NAME="zh_CN.UTF-8"
	LC_ADDRESS="zh_CN.UTF-8"
	LC_TELEPHONE="zh_CN.UTF-8"
	LC_MEASUREMENT="zh_CN.UTF-8"
	LC_IDENTIFICATION="zh_CN.UTF-8"
	LC_ALL=

将export LANG="en_US.UTF-8" 修改成与第一行一样的内容。


