---
layout: post
title:  使用artDialog时遇到的问题
description: artDialog是一个相当好用的对话框插件。用法可以参考：<a href="http://aui.github.io/artDialog/doc/index.html#api-show">http://aui.github.io/artDialog/doc/index.html#api-show</a>。  但我使用时遇到了下面的问题
date:   2014-12-29 16:49:39
categories: blog
tags: jQuery artdialog
---
artDialog是一个相当好用的对话框插件。用法可以参考：[http://aui.github.io/artDialog/doc/index.html#api-show](http://aui.github.io/artDialog/doc/index.html#api-show)。  
但我使用时遇到了下面的问题：  
在列表页面，针对每一条数据后面有一个设置按钮，单击设置按钮弹出一个设置窗口1。  
在设置窗口中有文件上传的部分，点击上传按钮会使用artDialog弹出一个提示窗口2（提示支持的文件类型，大小等，然后是一个选择文件的按钮）。  
现在的问题是：在设置窗口1点击上传按钮时，弹出的提示窗口2被1遮住了。  
我用的版本是4.1.2。  
百度呢，发现它有一个zIndex属性，不过呢这是一个全局的属性。也就是说呢，你第一个弹出窗口zIndex设置为99，  
那么后面的弹出窗口zIndex都是99.那么首先想到了针对设置窗口1和提示窗口2都设置zIndex。  
但仅仅这样扔有问题。  

第一次是成功的。但将子窗口关闭后，再次打开就被遮住了。由于zIndex是一个全局属性，因此在第一次打开子窗口时已经将zIndex设置为了新的值。  
再次打开时父窗口和子窗口的zIndex一样。父窗口大一些，将子窗口遮住了。   
那么，解决办法就是：在每次打开子窗口时改变zIndex，让他比父窗口大就行了。   

好吧，这样做了之后暂时没有什么问题了。   

但是，如果页面有可以input,select之类的，在你点了input,select后发现再点上传按钮又不行了！   
那我首先对设置窗口页面的所有input,select加了click事件处理，当click时将zIndex++。   
这样做之后，在你click input,select元素时，再点上传按钮没问题。   
但如果你改变了值就又不行了。      
所以，在失去焦点时，我将zIndex再次++.这样终于没问题了。   

**相关代码**：   

#####list页面：  
{% highlight javascript %}
function showSettings(obj) {
	var posno = $(obj).attr("posno");
	var smartid = $(obj).attr("smartid");
	var index = artindex();
	$.get("${ctx}/tsmartpos/posset/"+smartid,{posno:posno,shopid:shopid,ts:(new Date()).getTime()},function(data){
		   art.dialog({
			   id:'parentWindow',
			lock:true,
			opacity:0.3,
			title: '终端素材上传',
			width: '80%',
			top:'10%',
			drag:true, 
			zIndex:index,
			content: data,
			esc: true,
			init:function(){
				partdialog = this;
			}
		});
	  });
} 

function closeWin() {
	partdialog.close();
}
function artindex() {
	return 999;
}
{% endhighlight %}
		
#####设置窗口：  
{% highlight javascript %}
var zindex = parent.artindex();

//上传图片
function uploadImg(type){//上传类型  videourl：视频 , logourl：logo ,wxcodeurl： 二维码
	flag = type;
	
	$.get("${ctx}/tsmartset/gotouploadpic/"+type ,{ts:(new Date()).getTime()},function(data){
		zindex++;
		console.log('zindex:'+zindex);
		   art.dialog({
			   id:'imageart',
			lock:true,
			opacity:0.3,
			title: '上传图片',
			width: 470,
			left: '50%',
			top: '40%',
			drag:false,
			zIndex:zindex,
			content: data,
			esc: true,
			init: function(){
			   pic_artdialog = this;
			},
			close: function(){
			   //delpic(type)
			   pic_artdialog = null;
			   $('#face').imgAreaSelect({remove:true}); 
			}
		});
		pic_artdialog.show();
	  });
}

$("input").click(function(){
    zindex++;
}).blur(function(){
	zindex++;
});		
{% endhighlight %}

