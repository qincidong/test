---
layout: post
title:  Notepad++使用正则表达式替换
description: 在Notepad++中使用正则表达式实现批量替换文本。
date:   2014-12-30 10:12
categories: blog
tags: notepad++ 正则表达式
---
如：
{% highlight html %}
<li>Spring</li>
{% endhighlight %}

替换后为：
{% highlight html %}
<li><a href='#'>Spring</a></li>
{% endhighlight %}

查找的正则：
{% highlight html %}
<li>([^<]*)
{% endhighlight %}

替换的正则：
{% highlight html %}
<li><a href='#'>$1</a>
{% endhighlight %}