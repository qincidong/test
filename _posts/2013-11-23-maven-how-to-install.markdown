---
layout: post
title:  Maven的安装
description: Maven的安装
date:   2013-11-23 16:49:39
categories: blog
tags: maven
---
下载地址：[download](http://maven.apache.org/download.cgi)  
现在的最新版本是apache-maven-3.1.1-bin.zip  
下载之后解压缩，比如我的是F:\study\tools\apache-maven-3.0.5-bin\apache-maven-3.0.5。  
然后就是配置环境变量M2_HOME：  
新建用户变量：  
变量名：M2_HOME  
变量值：Maven解压缩的位置，比如我的是F:\study\tools\apache-maven-3.0.5-bin\apache-maven-3.0.5。

注意：Maven的运行依赖Java，需要安装JDK，并配置JAVA_HOME。  
打开CMD窗口，输入mvn -v，如果出现下图则说明安装成功了。  
![img](http://images.cnitblog.com/blog/548748/201311/23153623-669cc7ff10c74d0b890fb89a7f621a7a.png)
