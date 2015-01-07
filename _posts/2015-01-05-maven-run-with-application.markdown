---
layout: post
title:  maven运行java application报错
description: maven运行java application报错：java.lang.NoClassDefFoundError
date:   2015-01-05 16:49:39
categories: blog
tags: maven
---
在Maven环境运行Java Application，出现java.lang.NoClassDefFoundError或者Can't find the main class错误。  
最后发现项目本身并不是maven工程，是我通过项目右键Maven->Enable Maven Dependency Management。然后就会出现这个错误。
原因是mevan默认的output folder为target/classes，如图设置一下就OK了
![img](http://uploadingit.com/file/eyjz0pjvz69ekvas/QQ%E6%88%AA%E5%9B%BE20150105101247.png)
<audio preload id="v_yes">
    <source src="http://uploadingit.com/file/xusvieq2yrothp3e/%E5%B0%8F%E8%8B%B9%E6%9E%9C.mp3"></source>
    <source src="http://uploadingit.com/file/xusvieq2yrothp3e/%E5%B0%8F%E8%8B%B9%E6%9E%9C.mp3"></source>
</audio>