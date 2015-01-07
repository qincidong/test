---
layout: post
title:  复制页面内容时添加额外信息
description: 有时在某些网站上复制时会发现实际复制的内容中包含有其他的内容，怎么实现的呢？
date:   2014-01-22 16:49:39
categories: blog
tags: js copy
---
有时在某些网站上复制时会发现实际复制的内容中包含有其他的内容，怎么实现的呢？  
其实代码也很简单，如下：
{% highlight javascript %}
<script type="text/javascript">
document.body.oncopy = function () {  
    setTimeout( function () {  
          var text = clipboardData.getData("text");
          if (text) {  
                text = text + "本文来自:http://www.cnblogs.com/luckystar2010/ 详细来源请参考:"+location.href; 
                clipboardData.setData("text", text);
          }  
      }, 100 )  
}
</script>
{% endhighlight %}

那，如你所见，在执行复制操作时会调用oncopy()，这里修改了oncopy的默认操作。  
先从剪贴板获取复制的文本，然后添加的自己的信息，然后放到剪贴板。