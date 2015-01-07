---
layout: post
title:  为tomcat页面设置访问权限（BASIC认证）
description: 在web应用中，对页面的访问控制通常通过程序来控制，流程为：  登录 -> 设置session -> 访问受限页面时检查session是否存在，如果不存在，禁止访问  对于较小型的web应用，可以通过tomcat内置的访问控制机制来实现权限控制。  采用这种机制的好处是，程序中无需进行权限控制，完全通过对tomcat的配置即可完成访问控制。
date:   2014-12-19 16:49:39
categories: blog
tags: tomcat web安全 basic
---
在web应用中，对页面的访问控制通常通过程序来控制，流程为：  
登录 -> 设置session -> 访问受限页面时检查session是否存在，如果不存在，禁止访问  
对于较小型的web应用，可以通过tomcat内置的访问控制机制来实现权限控制。  采用这种机制的好处是，程序中无需进行权限控制，完全通过对tomcat的配置即可完成访问控制。

为了在tomcat页面设置访问权限控制，在项目的WEB-INFO/web.xml文件中，进行如下设置：
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
        <auth-method>BASIC</auth-method>
        <realm-name>TOMCAT BASIC认证</realm-name>
    </login-config>

    <security-role>
        <role-name>admin</role-name>
    </security-role>
    <security-role>
        <role-name>user</role-name>
    </security-role>
</web-app>
{% endhighlight %}
其中，<url-pattern>中指定受限的url，可以使用通配符*，通常对整个目录进行访问权限控制。  
<auth-constraint>中指定哪些角色可以访问<url-pattern>指定的url，在<role-name>中可以设置一个或多个角色名。  
使用的角色名来自tomcat的配置文件${CATALINA_HOME}/conf/tomcat-users.xml。  
<login-config>中设置登录方式，<auth-method>的取值为BASIC或FORM。如果为BASIC，浏览器在需要登录时弹出一个登录窗口。如果为FORM方式，需要指定登录页面和登录失败时的提示信息显示页面。  
在${CATALINA_HOME}/conf/tomcat-users.xml中添加下面的 角色和用户
{% highlight xml %}
<tomcat-users>
    <role rolename="admin"/>
    <role rolename="user"/>
    <user username="admin" password="123456" roles="admin,user"/>
    <user username="system" password="system" roles="admin"/>
    <user username="user" password="user" roles="user"/>
</tomcat-users>
{% endhighlight %}
如上，共2个角色admin和 user。  
3个用户，admin在角色admin和user下，system在admin角色下，user在user角色下。  
我的测试工程有2个文件webAuthentification/views/admin/index.jsp和webAuthentification/views/user/index.jsp，用来测试admin和user这2个角色。  
部署到tomcat，然后启动tomcat，并打开浏览器，输入http://localhost:8080/webAuthentification/views/admin/index.jsp，回车。  
此时浏览器会弹出一个窗口提示你输入用户名和密码，如下：  
![form](http://images.cnitblog.com/blog/548748/201312/19163734-2caf17ac0e1d48ae9e17fe8fc74a628e.png)

用户名和密码分别输入admin/123456，确定。  
访问admin/index.jsp页面Ok，页面显示：  
>Hello,Admin!!!   
>auth:Basic YWRtaW46MTIzNDU2  
>user:admin  
>pass:123456  

那访问user/index.jsp呢？ 
页面显示：  
>Hello,user!!!

因为admin用户同时拥有admin和user的角色，所以都可以访问。  
关闭浏览器，打开admin/index.jsp，输入system/system，  
页面显示：
>Hello,Admin!!! 
>auth:Basic c3lzdGVtOnN5c3RlbQ==
>user:system
>pass:system

再访问user/index.jsp:  
页面显示：
>HTTP Status 403 - Access to the requested resource has been denied
因为system用户没有user角色。

当然了，user用户就只能访问user/index.jsp。

附admin.jsp:
{% highlight jsp %}
<?xml version="1.0" encoding="GB18030" ?>
<%@ page language="java" contentType="text/html; charset=GB18030"
    pageEncoding="GB18030"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:f="http://java.sun.com/jsf/core" xmlns:h="http://java.sun.com/jsf/html">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=GB18030" />
<title>Admin Index Page</title>
</head>
<body>
    Hello,Admin!!!
    <br>
    <%
        String auth_user = null;
        String auth = request.getHeader("Authorization");
        String encoded = auth.substring(6);
        sun.misc.BASE64Decoder dec = new sun.misc.BASE64Decoder();
        String decoded = new String(dec.decodeBuffer(encoded));
        String[] userAndPass = decoded.split(":", 2);
        auth_user = userAndPass[0];
        session.setAttribute("user",auth_user);
    %>
    auth:<%=auth%><br>
    user:<%=userAndPass[0] %><br>
    pass:<%=userAndPass[1] %>
</body>
</html>
{% endhighlight %}
文章参考[http://www.blogjava.net/asktalk/archive/2005/07/23/8221.html](http://www.blogjava.net/asktalk/archive/2005/07/23/8221.html)
