---
layout: post
title:  离线安装Maven FOR Eclipse插件
description: 离线安装Maven FOR Eclipse插件
date:   2013-12-05 16:49:39
categories: blog
tags: maven eclipse tools
---
Eclipse插件有2种安装方式：  
<li>  1.在线安装，在Help->Software Updates，单击Add site，输入update url。</li>  
不过这种安装方式有一个缺点，那就是下载速度很慢，你要安装m2eclipse的话得等好久。  
<li>  2.离线安装。这种就是我们将需要的插件下载下来，跟Eclipse建立联系。  </li>

我们一般都用的是第一种方式，但是现在m2eclipse的插件更新地址连接不上了。刚好百度一下有人已经放出了m2eclipse的离线安装包。  
在百度中输入“Eclipse maven3 site:pan.baidu.com”。从百度云中查找m2eclipse的离线安装包。  
我们一般都是直接将插件丢到Eclipse的插件目录中，但是这样有一点不好，很多插件混合在一起，有时候连你自己都不知道哪些是哪个插件的。  
因此使用link的方式：  
在你的Eclipse目录中新建links和myplugins（这个名字随意）目录，将下载的离线安装包解压到myplugins中。  
在links目录中新建maven.link，用记事本打开。输入path=你解压的离线安装包的位置。保存。  
这样其实就已经安装Ok。  
不过，还需要做一步，因为maven使用的是jdk中的JAR，不是JRE中的，因此需要在eclipse.ini中首行加入：
{% highlight xml %}
-vm
C:/Program Files/Java/jdk1.6.0_20/bin/javaw.exe
{% endhighlight %}

Ok，这样打开Eclipse，到window->preferences，输入maven，你可以看到maven已经被集成进来了。
![tip](http://171.13.14.118/share.php?method=Share.download&cqid=be2fbc2fdaba27acfbf5d3916595a811&dt=51.92a0aace1b4485290bd0ea9ceeb11368&e=1420192242&fhash=ea177e46e209af134db52ce8771f28b339d5d8b4&fname=QQ%E6%88%AA%E5%9B%BE20141202145749.png&fsize=14770&nid=14175908454966895&scid=51&st=08bf1a21337ed5d9138ff01ae67d7f95&xqid=103776276)