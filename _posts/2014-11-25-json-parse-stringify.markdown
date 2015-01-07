---
layout: post
title:  JSON.parse()和JSON.stringify()
description: JSON.parse()和JSON.stringify()
date:   2014-11-25 16:49:39
categories: blog
tags: 前端 json parse stringify
---
parse用于从一个字符串中解析出json对象,如
{% highlight javascript %}
var str = '{"name":"huangxiaojian","age":"23"}'
{% endhighlight %}
结果：
{% highlight javascript %}
JSON.parse(str)
{% endhighlight %}
Object  
age: "23"  
name: "huangxiaojian"  
__proto__: Object  
注意：单引号写在{}外，每个属性名都必须用双引号，否则会抛出异常。

stringify()用于从一个对象解析出字符串，如
{% highlight javascript %}
var a = {a:1,b:2}
JSON.stringify(a)
{% endhighlight %}
结果：  
"{"a":1,"b":2}"

转载:[http://blog.csdn.net/wangxiaohu__/article/details/7254598](http://blog.csdn.net/wangxiaohu__/article/details/7254598)