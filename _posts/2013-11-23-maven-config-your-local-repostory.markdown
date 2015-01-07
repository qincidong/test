---
layout: post
title:  Maven配置本地仓库
description: Maven配置本地仓库
date:   2013-11-23 16:49:39
categories: blog
tags: maven
---
一般来说，在Maven安装成功后第一件事就是配置本地仓库。

因为Maven的默认仓库位置是~/.m2/repository。~代表“我的文档”。这样的话，在使用Maven构建时，所有依赖的Jar都是下载在这个位置。

当你的电脑要重新安装时这个位置就没啦~所以我们需要配置自己的本地仓库。

配置本地仓库其实很简单，在%M2_HOME%下一个settings.xml文件，使用UE打开我们会发现如下的注释：
{% highlight xml %}
<!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ~/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
{% endhighlight %}

这段注释告诉我们</localRepository>标签的作用是配置Maven依赖的文件保存的位置。且默认的位置是~/.m2/repository。

那假定我们自己的仓库位置是F:\study\maven\repository，那么我们需要做2件事情：

1.打开%M2_HOME%/settings.xml文件，在上面的这段注释后面增加<localRepository>F:\study\maven\repository</localRepository>；

2.复制settings.xml文件到F:\study\maven\repository。

Ok，这样就可以了。