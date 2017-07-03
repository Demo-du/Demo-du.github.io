---
layout: post
title: Unbuntu系统下MYSQL存储中文相关问题 
categories: Database
description: 
keywords: Database, Mysql
---

很多人在使用MYSQL的时候，存储中文会出现问题。根本原因是编码的问题。

按照如下操作，一般可以解决此类问题。注：笔者是在Ubuntu下做的测试，Windows下没试过。

## 1.进入MYSQL

mysql –u root –p然后输入密码


## 2.Mysql下输入show variables like 'character_set_%'

查看字符集设置

+--------------------------+----------------------------+
| Variable_name                 | Value                               |
+--------------------------+----------------------------+
| character_set_client         | utf8                                  |
| character_set_connection  | utf8                                 |
| character_set_database    | latin1                                |
| character_set_filesystem   | binary                               |
| character_set_results       | utf8                                  |
| character_set_server        | utf8                                  |
| character_set_system       | utf8                                 |
| character_sets_dir            | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+


通过查看，我们可以发现，character_set_database 的Value值为latin1；这就导致了数据库不能存储中文。


## 3.修改my.ini

此文件在笔者的电脑里位于/etc/mysql文件夹里，找到后可以进行修改，在后面加入：
[mysqld]
default-character-set=utf8
default-storage-engine=INNODB
sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
[mysql]
default-character-set=utf8

问题一般就可以解决了

## 4.还是不行的情况

理论上经过上一步应该就可以解决，但有时候却没什么用。这时候，可以在mysql命令下输入：set names utf8;

## 5.预防措施

此类问题也可以在建数据库时进行预防，在建表的时候最后加一句DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci，指定字符集。

