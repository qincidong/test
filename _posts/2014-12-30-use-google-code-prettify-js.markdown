---
layout: post
title:  使用 Google Code Prettify 实现代码高亮
description: 今天这篇文章主要讲述使用 google-code-prettify 来实现代码的高亮显示，以前我使用 highlight.js 来实现文章中代码的高亮显示。 prettify 非常小巧且配置简单，使用它来实现代码的高亮显示是个不错的选择。
date:   2014-12-30 16:49:39
categories: blog
tags: 代码高亮 js prettify
---
>今天这篇文章主要讲述使用 google-code-prettify 来实现代码的高亮显示，以前我使用 highlight.js 来实现文章中代码的高亮显示。 
>prettify 非常小巧且配置简单，使用它来实现代码的高亮显示是个不错的选择。下边我们简单看看 prettify.js 的使用方法：

###1.引入 jQuery 文件和 prettify.js 文件
{% highlight html %}
<script type="text/javascript"src="jquery-1.6.1.min.js"></script>
<script src="prettify.js" type="text/javascript"></script>
{% endhighlight %}

###2.调用 prettify.js 实现代码高亮
在 body 标签上添加调用方法，如下：
{% highlight html %}
<body onload="prettyPrint()">
</body>
{% endhighlight %}

将你需要高亮显示的代码片断放在&lt;pre&gt;标记里，如下：
{% highlight html %}
<pre class="prettyprint">
    @*你的代码片断*@
</pre>
{% endhighlight %}

###使用 jQuery 小技巧实现优化
上述方法可以实现代码的高亮，但每次手动为&lt;pre&gt;标签添加"prettyprint"类，显示有些麻烦。  
使用下边的代码片断来解决这个问题并替换掉 body 的"onload"的事件，实现分离：  
{% highlight js %}
$(window).load(function(){
     $("pre").addClass("prettyprint");
     prettyPrint();
})
{% endhighlight %}

到这我们应该已经成功使用 prettify.js 实现了代码的高亮显示，为了提高页面加载速度，我们应该将引用的 js 文件放置在底部。

下载地址：[http://code.google.com/p/google-code-prettify/][1]  
文本转载：[http://www.lidongkui.com/use-prettify-to-highlight-code][2]  
[1]:http://code.google.com/p/google-code-prettify/
[2]:http://www.lidongkui.com/use-prettify-to-highlight-code