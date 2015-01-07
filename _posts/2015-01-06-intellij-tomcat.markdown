---
layout: post
title: IntelliJ IDEA 12 与 Tomcat 集成并运行Web项目
description: IntelliJ IDEA 12 与 Tomcat 集成并运行Web项目
date:   2015-01-06 16:49:39
categories: blog
tags: intellij tomcat  tools
---
>打算从Eclipse换IDEA了，周六上午没事干来捣鼓捣鼓。配置基本改成自己习惯的了，并探索了一下IDEA的各种功能。   
>注：本文使用的是 IDEA12 U（试用30天）

下面分享下我在IDEA上直接把Web项目跑到Tomcat上的方法(跟Eclipse好像不太一样，有那么一点点小麻烦)具体步骤如下：

###1. 创建Web项目
创建Web项目的方法我就不多说了，参考：

用社区版 IDEA 和 普通版的 Eclipse 开发 Java Web 项目  
使用Idea社区版开发Web项目  
我直接使用从Eclipse导入过来的oschina项目。

###2. 配置你系统上的Tomcat到IDEA上
File-Settings-Application Servers-Add，设置你的Tomcat根目录。
![img](http://static.oschina.net/uploads/space/2012/1208/120702_BsUB_100267.jpg)
###3.为项目创建webapp Faclet
File-Project Structure-Modules，选中oschina并点击上面的+图标，在下拉菜单中选择Web创建一个Facelet，在右边根据你的项目修改配置：
![img](http://static.oschina.net/uploads/space/2012/1208/121911_85e3_100267.jpg)
###4. 创建Artifact
看到上一步图片最下方的提示让创建Artifact了吧，下面继续。切换到Artifacts Tab：
![img](http://static.oschina.net/uploads/space/2012/1208/123929_iT4f_100267.jpg)
在弹出的对话框中选择oschina：
![img](http://static.oschina.net/uploads/space/2012/1208/124135_iSpW_100267.jpg)
这部分好像没什么需要改的，直接点“OK”。

###5. 创建运行配置
Run-Edit Configurations，点上面的+为Tomcat添加配置：
![img](http://static.oschina.net/uploads/space/2012/1208/124601_bv5k_100267.jpg)
Server Tab的配置根据自己的需要修改，主要看Deployment Tab配置，还是点+：
![img](http://static.oschina.net/uploads/space/2012/1208/124843_nsrE_100267.jpg)
IDEA自动为我选择了前面创建的Artifact，直接点确定就可以了。
![img](http://static.oschina.net/uploads/space/2012/1208/125041_UzWJ_100267.jpg)
###6. 把Tomcat跑起来吧
看到下图中的Run按钮里已经有一个Tomcat图标了吧，点一下右边的运行按钮：
![img](http://static.oschina.net/uploads/space/2012/1208/125443_d9fm_100267.jpg)
不出意外的话，在IDE最下面的Run Tab里可以看到Tomcat启动时输出的信息。
![img](http://static.oschina.net/uploads/space/2012/1208/130210_GV7O_100267.jpg)
【全文完】

转载：[http://my.oschina.net/tsl0922/blog/94621](http://my.oschina.net/tsl0922/blog/94621)