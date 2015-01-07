---
layout: post
title:  绝对定位元素的水平垂直居中
description: 绝对定位元素的水平垂直居中
date:   2013-12-03 16:49:39
categories: blog
tags: div 居中
---
我知道的是这种方式：
{% highlight html%}
.ele {
    width:600px;
    height:500px;
    position:absolute;
    left:50%;
    top:50%;
    margin-top:-250px;
    margin-left:-300px;
}
{% endhighlight %}

这种方式倒也好理解。  
width:600px,height:500px:从浏览器的左上角（坐标[0,0]）开始画出一个宽度为600像素，高度为500像素的矩形。  
left:50%:将元素向右平移50%，此时元素并不在正中间，此时是元素的最左边是整个屏幕的正中间，看起来元素是偏向右的。  
top:50%:将元素向下平移50%，此时元素在垂直方向也不是正中间，最上面的一边是在屏幕的正中间，看起来是偏向下的。  
其实，你可能也看出来了，要使元素水平居中显示，要将元素水平向左平移元素本身宽度的一半，要使元素垂直居中，则将元素向上平移元素本身高度的一半。  
这样就可以了。所以通过margin-top和margin-left来分别调整向上、左平移元素本身高度的一半、宽度的一半，以使元素达到绝对居中。 

在http://www.zhangxinxu.com看到了另外一种方式：margin:auto实现绝对定位元素的居中
{% highlight html %}
.element {
    width: 600px; height: 400px;
    position: absolute; left: 0; top: 0; right: 0; bottom: 0;
    margin: auto;    /* 有了这个就自动居中了 */
}
{% endhighlight %}

不过这个IE浏览器只支持IE8以上的，Firefox和Chrome是支持的。
