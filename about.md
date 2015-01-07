---
layout: default
title: about
---
<div id="content" class="aboutMe">
<form class="page-loc" method="GET" action="/search">
	<span style="float:right"><input type="text" class="web-search" name ="q" value="站内搜索" /><a href="/atom.xml" class="page-rss" style="margin-left: 20px;">订阅</a></span>
  	luckystar's blog » 关于我
</form>
<dl class="aboutDl">
	<dd><strong>weibo:</strong><a href="http://weibo.com/luckystar2008" target="_blank">@luckystar</a></dd>
	<dd><strong>email:</strong><a href="mailto:qincidong@qq.com">qincidong@qq.com</a></dd>
	<dd><strong>自述:</strong>苦逼的JAVA程序员，对Java，数据库，jQuery等感兴趣。</dd>

	<dt>关于博客</dt>
	<dd>所有文章非特别说明皆为原创，遵循的协议为「<a href="http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh" target="_blank">署名-非商业性使用-相同方式共享</a>」，由于文章表述或者内容可能存在诸多错误，所以部分内容会作修改，为保证转载信息与源同步，转载请注明文章出处！谢谢合作 :）</dd>

	<dt>好友链接</dt>
	<dd>
        <div class="friend-link">
            <ul>
			</li>
			<a href="http://blog.csdn.net/qincidong" title="http://blog.csdn.net/qincidong" target="_blank">我的CSDN博客</a>
			</li>
			<li>
            <a href="http://www.cnblogs.com/luckystar2010/" title="http://www.cnblogs.com/luckystar2010/" target="_blank">我的博客园</a>
			</li>
			</ul>
        </div>
   </dd>
</dl>
{% include disqus.snippet %}
<div class="footer">
    <small>Powered by <a href="https://github.com/mojombo/jekyll">Jekyll</a> | Copyright 2013 - 2099 Designed by <a href="/about.html">luckystar</a> | <span class="label label-info">{{site.time | date:"%Y-%m-%d %H:%M:%S"}}</span></small>
</div>
</div>
<script type="text/javascript">
$(function(){
	$('#disqus_container .comment').trigger('click');
});
</script>
