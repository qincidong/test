---
layout: post
title:  限制必须使用微信打开网页
description: 限制必须使用微信打开网页
date:   2014-11-26 16:49:39
categories: blog
tags: 微信
---
####限制必须使用微信打开页面   
在页面JS中加入下面的代码：  

{% highlight javascript %}
function is_weixin(){
	var ua = navigator.userAgent.toLowerCase();
	if(ua.match(/MicroMessenger/i)=="micromessenger") {
		return true;
	 } else {
		return false;
	}
}

if (!is_weixin()) {
	window.location.href = 'https://open.weixin.qq.com/connect/oauth2/authorize?appid=wxdf3f22ebfe96b912&redirect_uri=xxx&response_type=code&scope=snsapi_base&state=hyxt#wechat_redirect';
}
{% endhighlight %}

微信内置一个浏览器，userAgent为micromessenger。