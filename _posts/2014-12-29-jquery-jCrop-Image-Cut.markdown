---
layout: post
title:  jQuery图片裁剪问题记录
description: jQuery图片裁剪在手机上裁剪图片，发现偶尔图片上没有出现裁剪框。最后发现可能是图片在JS之前加载的原因导致的。
date:   2014-12-29 16:49:39
categories: blog
tags: jQuery 图片 jquery.Jcrop
---
###问题描述

这个是给周大福做的LOMO打印机需求，在手机上裁剪图片保存时报下面的错误：  
{% highlight java %}
/wxsmart/cutSuccess?photourl=/upload/picmsg/1423/4599111449.jpg&memberid=572&x=0&y=0&w=0&h=0&wh=149&ww=224] INFO  com.msp.wxcrm.dao.TsmartSetDao - findByProperty
java.lang.IllegalArgumentException: Width (800) and height (0) cannot be <= 0
	at java.awt.image.DirectColorModel.createCompatibleWritableRaster(DirectColorModel.java:999)
	at java.awt.image.BufferedImage.<init>(BufferedImage.java:312)
	at com.msp.wxcrm.service.EApiManager.cutImg(EApiManager.java:6009)
	at com.msp.wxcrm.service.EApiManager$$FastClassByCGLIB$$cefd2b7d.invoke(<generated>)
	at net.sf.cglib.proxy.MethodProxy.invoke(MethodProxy.java:149)
	at org.springframework.aop.framework.Cglib2AopProxy$DynamicAdvisedInterceptor.intercept(Cglib2AopProxy.java:617)
	at com.msp.wxcrm.service.EApiManager$$EnhancerByCGLIB$$8825fc69.cutImg(<generated>)
	at com.msp.wxcrm.controller.WxSmartController.cutSuccess(WxSmartController.java:161)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
	at java.lang.reflect.Method.invoke(Method.java:597)
	at org.springframework.web.bind.annotation.support.HandlerMethodInvoker.invokeHandlerMethod(HandlerMethodInvoker.java:175)
	at org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter.invokeHandlerMethod(AnnotationMethodHandlerAdapter.java:421)
	at org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter.handle(AnnotationMethodHandlerAdapter.java:409)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:774)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:719)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:644)
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:549)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:689)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:802)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:252)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:173)
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:77)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:76)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:202)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:173)
	at org.springframework.orm.hibernate3.support.OpenSessionInViewFilter.doFilterInternal(OpenSessionInViewFilter.java:198)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:76)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:202)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:173)
	at javacommon.filter.LoggerMDCFilter.doFilterInternal(LoggerMDCFilter.java:39)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:76)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:202)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:173)
	at cn.org.rapid_framework.web.scope.FlashFilter.doFilterInternal(FlashFilter.java:28)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:76)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:202)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:173)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:88)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:76)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:202)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:173)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:213)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:178)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:126)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:105)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:107)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:148)
	at org.apache.coyote.http11.Http11Processor.process(Http11Processor.java:869)
	at org.apache.coyote.http11.Http11BaseProtocol$Http11ConnectionHandler.processConnection(Http11BaseProtocol.java:664)
	at org.apache.tomcat.util.net.PoolTcpEndpoint.processSocket(PoolTcpEndpoint.java:527)
	at org.apache.tomcat.util.net.LeaderFollowerWorkerThread.runIt(LeaderFollowerWorkerThread.java:80)
	at org.apache.tomcat.util.threads.ThreadPool$ControlRunnable.run(ThreadPool.java:684)
	at java.lang.Thread.run(Thread.java:619)
{% endhighlight %}

页面没有显示截图的边框。  
裁剪的代码如下：  
{% highlight jsp %}
	<%@ page language="java" pageEncoding="UTF-8"%>
<%@ include file="/commons/taglibs.jsp" %>
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<meta content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" name="viewport">
		<meta content="yes" name="apple-mobile-web-app-capable">
		<meta content="black" name="apple-mobile-web-app-status-bar-style">
		<meta name="format-detection" content="telephone=no">
		<title>照片裁剪</title>
		<link href="${fileUrlPrx}/skins/css/wap_index1.css" rel="stylesheet" type="text/css">
		<link href="${fileUrlPrx}/skins/css/wap_jquery.Jcrop.min.css" rel="stylesheet" type="text/css">
		<script type="text/javascript">
			
			function initCut(){
				if(navigator.userAgent.indexOf("windows Phone")>-0){
					setTimeout("doCut()",1000);  
				}else{
					doCut();
				}
			}		
			
			function doCut(){
				function updateCoords(c){
					$('#x').val(c.x);
					$('#y').val(c.y);
					$('#w').val(c.w);
					$('#h').val(c.h);
				};
				var wh = $("#clipArea").height();
				var ww = $("#clipArea").width();
				var cutwidth = wh>ww?ww:wh ;
				$('#cropbox').Jcrop({
					
					minSize: [cutwidth,cutwidth],
					maxSize:[cutwidth ,cutwidth],  
					setSelect: [0,0,cutwidth, cutwidth], 
					aspectRatio: 1,
					onSelect: updateCoords
				});
				function checkCoords(){
					if(parseInt($.trim($('#w').val())) > 0){
						return true;
					};
					alert('请裁剪你所需要的照片。'); 
					return false;
				};
				$('.clipBtn a').click(function(){
					checkCoords();
				});
			}
			function cutPic(){
				var x = $("#x").val();
				var y = $("#y").val();
				var w = $("#w").val();
				var h = $("#h").val();
				var wh = $("#clipArea").height();
				var ww = $("#clipArea").width();
				window.location.href="${fileUrlPrx}/wxsmart/cutSuccess?photourl=${photourl}&memberid=${tplog.memberid}&x="+x+"&y="+y+"&w="+w+"&h="+h+"&wh="+wh+"&ww="+ww;
			}
		</script>
	</head>
	<body>
		<div class="clipBox">
			<div class="flower-r"></div> 
			<div class="flower-l"></div>
			<div class="clipArea" id="clipArea" style="width: 70%;">
				<c:if test="${ errMsg != null }">
					<font style="color: red;font-weight: 700;font-size: 16px;">${errMsg}</font>
				</c:if>
				<c:if test="${ errMsg == null }">
					<div class="clipShow">
						<img id="cropbox" src="${fileUrlPrx}${tplog.photourl}" onload="initCut()"/>					
					</div>
				</c:if>
			</div>			
		</div>
		<c:if test="${ errMsg == null }">
			<div class="clipBtn">
				<a href="javascript:void(0)" onclick="cutPic();">确定裁剪</a>
			</div>
		</c:if>
		<input type="hidden" id="x" name="x" value="0"/>
		<input type="hidden" id="y" name="y" value="0"/>
		<input type="hidden" id="w" name="w" value="0"/>
		<input type="hidden" id="h" name="h" value="0"/>
		<input type="hidden" id="memberid" name="memberid" value="${tplog.memberid }"/>
		<div class="copyRight">技术支持：XXXX</div>
		<script src="${fileUrlPrx}/scripts/wap/jquery.min.js"></script>
		<script src="${fileUrlPrx}/scripts/wap/jquery.Jcrop.min.js"></script>
	</body>
</html>
{% endhighlight %}
使用的是jquery.Jcrop插件，而且出现这个问题只有在首次关注使用时才会有。我也看了Jcrop的API，都没问题。  
最后发现JS的顺序可能导致了这个问题。如上，图片在jquery.js,jquery.jscrop.min.js之前加载完毕。  
图片加载完毕之后就会执行doCut方法，如果这个时候2个JS还没有加载完毕，就会导致JS错误。这样的话肯定显示不了裁剪的边框，所以w,h都为0.  

jQuery Jcrop 图像裁剪使用参考：[jQuery Jcrop 图像裁剪](http://code.ciaoca.com/jquery/jcrop/)