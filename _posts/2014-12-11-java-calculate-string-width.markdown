---
layout: post
title: java计算字符串的宽度和高度
description: java计算字符串的宽度和高度
date:   2014-12-11 16:49:39
categories: blog
tags: java String
---
{% highlight java %}
//g对象为一个Graphics
FontMetrics fm = g.getFontMetrics ()；          
int strWidth = fm.stringWidth ("Registering plug-ins……")；          
int strHeight = fm.getHeight ()；  
{% endhighlight %}

转载：[http://sbje5201314.blog.163.com/blog/static/280338620086332426488/](http://sbje5201314.blog.163.com/blog/static/280338620086332426488/)