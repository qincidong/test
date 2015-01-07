---
layout: post
title:  无提示关闭弹出窗口
description: 无提示关闭弹出窗口
date:   2013-11-29 16:49:39
categories: blog
tags: js window close
---
主要代码：
{% highlight javascript %}
window.open('','_parent','');
window.close();
{% endhighlight %}

一个父子窗口的例子 ：  
**parent.html:**
{% highlight html %}
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN">
<html>
<head>
<title> 父窗口</title>
<meta name="Generator" content="EditPlus">
<meta name="Author" content="">
<meta name="Keywords" content="">
<meta name="Description" content="">
<script language="JavaScript">
<!--

    function popupWin() {
        window.open('child.html','','');
    }
    function closeWindow() {
        window.open('','_parent','');
        window.close();
    }
//-->
</script>
</head>

<body>
    <input type="button" value="弹出子窗口" onclick="popupWin();">
    <input type="button" value="关闭窗口" onclick="closeWindow();">
</body>
</html>
{% endhighlight %}

**child.html:**
{% highlight html %}
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN">
<html>
<head>
<title> New Document </title>
<meta name="Generator" content="EditPlus">
<meta name="Author" content="">
<meta name="Keywords" content="">
<meta name="Description" content="">
</head>

<body>
    <script language="JavaScript">
    <!--
        function closeWindow() {
            window.open('','_parent','');
            window.close();
        }
    //-->
    </script>
    <input type="button" value="关闭窗口" onclick="closeWindow();">
    
</body>
</html>
{% endhighlight %}

如果是Firefox浏览器的话，需要将dom.allow_scripts_to_close_window设置为true。  
具体就是打开一个新的标签页，在地址栏输入about:config，回车。  
然后找到dom.allow_scripts_to_close_window，双击设置为true。