---
layout: post
title: 配置Struts2出现The request resource….is not available问题 
categories: javaEE
description: 
keywords: java,Struts2
---

早上兴致勃勃的打算写一个Struts2的小项目，添加库，各种配置后，然后测试，结果。。。。。。

![OpenGrok Search and Browse](/images/posts/java/error1.png)

心情几乎是崩溃的。。。不应该啊，以前没出现过这种情况。

既然出现了，开始查找原因。首先问了一下度娘，度娘给的回复是：404问题一般是路径的原因。检查了一下，路径并没有任何问题。

然后怀疑自己配置有问题，或者java程序写的有问题。于是，挨个检查，查了好久，也没查出来，一气之下，把Struts怀疑自己配置有问题，或者java程序写的有问题。于是，挨个检查，查了好久，也没查出来，一气之下，把Struts.xml中的配置全删掉，只剩下开头还有<struts></struts>了，这下虽然这个程序没什么用了，但应该不报错了吧。。。。一运行，还是报错。。。。。

我意识到，既然删掉配置信息也报错，那可能就不是配置的问题，web.xml关于Struts的配置信息很短，基本不会出错，而且检查了好多遍。所以，很大可能是添加的lib里面有问题。仔细检查了一下，果然，多加了一个Struts2-spring-plugin….这个文件是用来整合Struts2和spring的，如果不用整合，只用Struts2，那么就会报错。问题虽小，但希望引以为戒。
