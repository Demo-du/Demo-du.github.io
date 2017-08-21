---
layout: post
title: Redis问题：MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error. 
categories: Redis Database
description: 
keywords: 
---


今天操作redis，在连接Redis的时候出现了这种问题：

MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error.

翻译：Redis被配置为保存数据库快照，但它目前不能持久化到硬盘。用来修改集合数据的命令不能用。请查看Redis日志的详细错误信息。

问题原因分析：

强制关闭Redis快照导致不能持久化。

解决方案：

将stop-writes-on-bgsave-error设置为no

按照以上方法问题得到了解决，但后来又看到一篇博文，http://www.cnblogs.com/qq78292959/p/3994349.html，感激这种方法更安全一点。
