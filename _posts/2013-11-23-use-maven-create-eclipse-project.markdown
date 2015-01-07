---
layout: post
title:  使用Eclipse创建Maven项目
description: 使用Eclipse创建Maven项目
date:   2013-11-23 16:49:39
categories: blog
tags: maven eclipse
---
首先要安装maven for eclipse的插件m2eclipse。  
地址：[m2ecipse](http://www.eclipse.org/m2e/download/)
打开Eclipse，在Eclipse Marketplace中找m2eclipse
![img](http://marketplace.eclipse.org/sites/all/themes/nova/images/helpMarketplace.png)
在Find输入框输入m2eclipse，然后回车，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23200811-5eb95231e001445a91d0b03b28cf8712.png)
我们要安装的插件式第1个，因为我已经安装了，所以显示的是update和Uninstall。  
如果没安装过，显示的是Install，然后单击这个按钮。
![img](http://images.cnitblog.com/blog/548748/201311/23200915-a85b35f1c34949a28c4a3d51303fe8c3.png)
然后单击”Next“按钮， 
![img](http://images.cnitblog.com/blog/548748/201311/23200938-bc5893cb0c044e56b4e26d916a52a067.png)
然后选择”I accept the terms of the license agreements“，然后单击”Finish“按钮。  
然后Eclipse就会下载并安装，安装完毕后会提示你需要重启Eclipse才能生效，重启Eclipse就可以了。  
这样我们的m2eclipse插件就安装完毕了。
 

安装完m2eclipse插件还需要进行一些设置。  
打开Windows->Preferences，找到Maven。  
首先要修改的是Installations，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23201328-30a7e6bd18454d4e898e1285581f0e08.png)
它自带了一个，不过不能用，我们要指定自己的Maven的位置。单击Add按钮，然后找到我们自己的Maven位置。  
如上面我的是F:\study\tools\apache-maven-3.0.5-bin\apache-maven-3.0.5\。

然后要修改的是User settings，如图是它的默认配置。
![img](http://images.cnitblog.com/blog/548748/201311/23201546-40bc0990388c4f0bbe18ebe8f2c64c77.png)

你可以看到，它默认的本地仓库位置是我的文档中的.m2，我们需要修改为我们自己在settings.xml中配置的LocalRepostory指定的位置。  
我的本地仓库位置是：F:/study/maven/repository。如图：
![img](http://images.cnitblog.com/blog/548748/201311/23201806-b46f089cedce4967bd8dde142cf2b309.png)
然后单击Apply,Ok按钮。  
Ok，到此Maven的配置就结束了。下面我们用Eclipse来创建一个Maven项目。  
New->Others->Maven->Maven Project，然后选择”create a simple project(skip archetype selection)“。  
然后就是输入groupId,artifactId,version,package等，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23202224-2e59b074cdd043439acd4e5ca1dcdb0c.png)
然后单击”Finish“按钮。  
创建的项目结构是这样的：
![img](http://images.cnitblog.com/blog/548748/201311/23202410-ce65585a85c5458dbe4b75d4a3b1ad80.png)
Java的源代码就放在src/main/java中，测试类源代码放在src/test/java中，项目中用到的配置文件在src/main/resources中，单元测试用到的资源文件在src/test/resources中。  
默认创建的pom.xml中是没有任何依赖的，如果我们要添加依赖则双击打开pom.xml文件，任何单击”Dependencies“选项卡，单击Add按钮添加依赖，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23203552-c79cf224d21f4b15bb067879627502b4.png)
任何单击Ok，就添加了junit的依赖。  
这是我们知道groupId和artifactId的情况，如果我们不知道呢？  
我们可以到中央仓库去查找。  
中央仓库在哪儿？  
在%M2_HOME%/lib/maven-model-builder-3.0.5.jar中有一个pom-4.0.0.xml，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23203759-043f5bc373db47e5ab3063497a776079.png)
打开pom-4.0.0.xml文件，在id位central的repostory中的url就是中央仓库的位置，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23203940-0084e0bb766a41c4959f2f125c5906df.png)
Ok，假如我们现在要使用log4j，首先打开http://central.maven.org/maven2/
![img](http://images.cnitblog.com/blog/548748/201311/23204041-c38d7d88eeba41359a6a85d8ff920c08.png)
如图，首先告诉我们不可用，在http://search.maven.org查看目录内容。Ok我们点进去。  
在输入框中输入log4j，然后查找，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23204301-db9e210899554694a1cdb61b759f0f34.png)
如上图，列出了很多个版本的log4j，我们选择1.2.17这个版本。
![img](http://images.cnitblog.com/blog/548748/201311/23204440-b29a08fff49c413090ae3b238ffc08e9.png)
如上图，在Dependency Information中列出了依赖定义，复制粘贴到pom.xml中就行了。  
现在在代码中就可以使用log4j了，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23204642-5b6aafcb4fb24a81b109436ca72dc046.png)
如何构建maven项目呢？  
右键pom.xml，有一系列的菜单比如maven build,mavn clean,maven install,maven test等等。