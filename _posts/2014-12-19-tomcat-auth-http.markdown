---
layout: post
title:  为Tomcat页面设置访问权限（HTTP）
description: 为Tomcat页面设置访问权限（HTTP）
date:   2014-12-19 16:49:39
categories: blog
tags: tomcat web安全 http
---
web.xml:
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="WebApp_ID" version="2.5">
    <display-name>webAuthentification</display-name>
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>admin</web-resource-name>
            <url-pattern>/views/admin/*</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>admin</role-name>
        </auth-constraint>
    </security-constraint>
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>user</web-resource-name>
            <url-pattern>/views/user/*</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>user</role-name>
        </auth-constraint>
    </security-constraint>
    <login-config>
        <auth-method>FORM</auth-method>
        <realm-name>TOMCAT FORM认证</realm-name>
        <form-login-config>
            <form-login-page>/views/common/login.jsp</form-login-page>
            <form-error-page>/views/common/error.jsp</form-error-page>
        </form-login-config>
    </login-config>
    <security-role>
        <role-name>admin</role-name>
    </security-role>
    <security-role>
        <role-name>user</role-name>
    </security-role>
</web-app>
{% endhighlight %}

与BASIC认证不同的主要是<login-config/>这一块，修改为FORM认证，并指定响应的登陆页面和登陆失败后的页面。  
要注意登陆页面中用户名的name必须是j_username，密码的name必须是j_password，Form的action必须是j_security_check.  
例子：
{% highlight jsp %}
<?xml version="1.0" encoding="GB18030" ?>
<%@ page language="java" contentType="text/html; charset=GB18030"
    pageEncoding="GB18030"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
    xmlns:f="http://java.sun.com/jsf/core"
    xmlns:h="http://java.sun.com/jsf/html">
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=GB18030" />
		<title>Login Page</title>
	</head>
<body>
	<form method=post action='<%=response.encodeURL("j_security_check")%>'>
		<table border="0" cellspacing="5">
			<tr>
				<th align="right">Username:</th>
				<td align="left"><input type="text" name="j_username"/></td>
			</tr>
			<tr>
				<th align="right">Password:</th>
				<td align="left"><input type="password" name="j_password"/></td>
			</tr>
			<tr>
				<td align="right"><input type="submit" value="Log In"/></td>
				<td align="left"><input type="reset"/></td>
			</tr>
		</table>
	</form>
</body>
</html>
{% endhighlight %}
这样就ok了，在访问一个<web-resource-collection/>指定的受保护的资源时，会先跳转到登陆页面登陆，登陆失败则调整到失败页面；登陆成功则访问登陆之前的页面。  
注意，如果是Eclipse集成的TOMCAT则认证总是失败，必须是启动安装的tomcat的bin/start.bat。