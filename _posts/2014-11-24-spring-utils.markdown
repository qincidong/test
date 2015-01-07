---
layout: post
title:  Spring常用工具类
description: Spring常用工具类~~~
date:   2014-11-24 16:49:39
categories: blog
tags: Spring
---
[Spring 的优秀工具类盘点](http://www.ibm.com/developerworks/cn/java/j-lo-spring-utils1/index.html)
###文件资源操作
Spring 定义了一个 org.springframework.core.io.Resource 接口，Resource 接口是为了统一各种类型不同的资源而定义的，Spring 提供了若干 Resource 接口的实现类，  
这些实现类可以轻松地加载不同类型的底层资源，并提供了获取文件名、URL 地址以及资源内容的操作方法   
####访问文件资源   
* 通过 FileSystemResource 以文件系统绝对路径的方式进行访问；    
* 通过 ClassPathResource  以类路径的方式进行访问；   
* 通过 ServletContextResource 以相对于 Web 应用根目录的方式进行访问。   
{% highlight java %}
package com.baobaotao.io;
import java.io.IOException;
import java.io.InputStream;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
public class FileSourceExample {
    public static void main(String[] args) {
        try {
            String filePath =
            "D:/masterSpring/chapter23/webapp/WEB-INF/classes/conf/file1.txt";
            // ① 使用系统文件路径方式加载文件
            Resource res1 = new FileSystemResource(filePath);
            // ② 使用类路径方式加载文件
            Resource res2 = new ClassPathResource("conf/file1.txt");
            InputStream ins1 = res1.getInputStream();
            InputStream ins2 = res2.getInputStream();
            System.out.println("res1:"+res1.getFilename());
            System.out.println("res2:"+res2.getFilename());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}
在获取资源后，您就可以通过 Resource 接口定义的多个方法访问文件的数据和其它的信息  
getFileName() 获取文件名，  
getFile() 获取资源对应的 File 对象，  
getInputStream() 直接获取文件的输入流。  
createRelative(String relativePath) 在资源相对地址上创建新的资源。  
在 Web 应用中，您还可以通过 ServletContextResource 以相对于 Web 应用根目录的方式访问文件资源  
Spring 提供了一个 ResourceUtils 工具类，它支持“classpath:”和“file:”的地址前缀 ，它能够从指定的地址加载文件资源。  

{% highlight java %}
File clsFile = ResourceUtils.getFile("classpath:conf/file1.txt");  
System.out.println(clsFile.isFile());
String httpFilePath = "file:D:/masterSpring/chapter23/src/conf/file1.txt";
File httpFile = ResourceUtils.getFile(httpFilePath);
{% endhighlight %}

####文件操作  
在使用各种 Resource 接口的实现类加载文件资源后，经常需要对文件资源进行读取、拷贝、转存等不同类型的操作。    

#####FileCopyUtils  
它提供了许多一步式的静态操作方法，能够将文件内容拷贝到一个目标 byte[]、String 甚至一个输出流或输出文件中。   
{% highlight java %}
package com.baobaotao.io;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileReader;
import java.io.OutputStream;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.util.FileCopyUtils;
public class FileCopyUtilsExample {
    public static void main(String[] args) throws Throwable {
        Resource res = new ClassPathResource("conf/file1.txt");
		// ① 将文件内容拷贝到一个 byte[] 中
        byte[] fileData = FileCopyUtils.copyToByteArray(res.getFile());
        // ② 将文件内容拷贝到一个 String 中
        String fileStr = FileCopyUtils.copyToString(new FileReader(res.getFile()));
        // ③ 将文件内容拷贝到另一个目标文件
        FileCopyUtils.copy(res.getFile(),
        new File(res.getFile().getParent()+ "/file2.txt"));
		// ④ 将文件内容拷贝到一个输出流中
        OutputStream os = new ByteArrayOutputStream();
        FileCopyUtils.copy(res.getInputStream(), os);
    }
}
static void copy(byte[] in, File out)     将 byte[] 拷贝到一个文件中
static void copy(byte[] in, OutputStream out)     将 byte[] 拷贝到一个输出流中
static int copy(File in, File out)     将文件拷贝到另一个文件中
static int copy(InputStream in, OutputStream out)     将输入流拷贝到输出流中
static int copy(Reader in, Writer out)     将 Reader 读取的内容拷贝到 Writer 指向目标输出中
static void copy(String in, Writer out)     将字符串拷贝到一个 Writer 指向的目标中属性文件操作  
Spring 提供的 PropertiesLoaderUtils 允许您直接通过基于类路径的文件 地址加载属性资源  
package com.baobaotao.io;
import java.util.Properties;
import org.springframework.core.io.support.PropertiesLoaderUtils;
public class PropertiesLoaderUtilsExample {
    public static void main(String[] args) throws Throwable {   
    // ① jdbc.properties 是位于类路径下的文件
  Properties props = PropertiesLoaderUtils.loadAllProperties("jdbc.properties");
  System.out.println(props.getProperty("jdbc.driverClassName"));
    }
}
{% endhighlight %}

#####特殊编码的资源  
当您使用 Resource 实现类加载文件资源时，它默认采用操作系统的编码格式。  
如果文件资源采用了特殊的编码格式（如 UTF-8），则在读取资源内容时必须事先通过 EncodedResource 指定编码格式，否则将会产生中文乱码的问题。  
{% highlight java %}
package com.baobaotao.io;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.EncodedResource;
import org.springframework.util.FileCopyUtils;
public class EncodedResourceExample {
        public static void main(String[] args) throws Throwable  {
			Resource res = new ClassPathResource("conf/file1.txt");
            // ① 指定文件资源对应的编码格式（UTF-8）
			EncodedResource encRes = new EncodedResource(res,"UTF-8");
            // ② 这样才能正确读取文件的内容，而不会出现乱码
			String content  = FileCopyUtils.copyToString(encRes.getReader());
            System.out.println(content); 
    }
}
{% endhighlight %}

访问 Spring 容器，获取容器中的 Bean，使用 WebApplicationContextUtils 工具类  
{% highlight java %}
ServletContext servletContext = request.getSession().getServletContext();
WebApplicationContext wac = WebApplicationContextUtils. getWebApplicationContext(servletContext);
{% endhighlight %}

####Spring 所提供的过滤器和监听器   
#####延迟加载过滤器  
Hibernate 允许对关联对象、属性进行延迟加载，但是必须保证延迟加载的操作限于同一个 Hibernate Session 范围之内进行。  
如果 Service 层返回一个启用了延迟加载功能的领域对象给 Web 层，当 Web 层访问到那些需要延迟加载的数据时，由于加载领域对象的 Hibernate Session 已经关闭，这些导致延迟加载数据的访问异常。  
Spring 为此专门提供了一个 OpenSessionInViewFilter 过滤器，它的主要功能是使每个请求过程绑定一个 Hibernate Session，即使最初的事务已经完成了，也可以在 Web 层进行延迟加载的操作。  
{% highlight xml %}
<filter>
    <filter-name>hibernateFilter</filter-name>
    <filter-class>
    org.springframework.orm.hibernate3.support.OpenSessionInViewFilter
    </filter-class>
</filter>
<filter-mapping>
    <filter-name>hibernateFilter</filter-name>
    <url-pattern>*.html</url-pattern>
</filter-mapping>
{% endhighlight %}

#####中文乱码过滤器  
{% highlight xml %}
<filter>
<filter-name>encodingFilter</filter-name>
<filter-class>
	org.springframework.web.filter.CharacterEncodingFilter ① Spring 编辑过滤器
</filter-class>
<init-param> ② 编码方式
	<param-name>encoding</param-name>
	<param-value>UTF-8</param-value>
</init-param>
<init-param> ③ 强制进行编码转换
	<param-name>forceEncoding</param-name>
	<param-value>true</param-value>
</init-param>
</filter>
<filter-mapping> ② 过滤器的匹配 URL
	<filter-name>encodingFilter</filter-name>
	<url-pattern>*.html</url-pattern>
</filter-mapping>
{% endhighlight %}

一般情况下，您必须将 Log4J 日志配置文件以 log4j.properties 为文件名并保存在类路径下。  
Log4jConfigListener 允许您通过 log4jConfigLocation Servlet 上下文参数显式指定 Log4J 配置文件的地址，如下所示：  
{% highlight xml %}
① 指定 Log4J 配置文件的地址
<context-param>
    <param-name>log4jConfigLocation</param-name>
    <param-value>/WEB-INF/log4j.properties</param-value>
</context-param>
② 使用该监听器初始化 Log4J 日志引擎
<listener>
    <listener-class>
    org.springframework.web.util.Log4jConfigListener
    </listener-class>
</listener>
Introspector 缓存清除监听器,防止内存泄露
<listener>
    <listener-class>
    org.springframework.web.util.IntrospectorCleanupListener
    </listener-class>
</listener>
{% endhighlight %}
一些 Web 应用服务器（如 Tomcat）不会为不同的 Web 应用使用独立的系统参数，也就是说，应用服务器上所有的 Web 应用都共享同一个系统参数对象。  
这时，您必须通过 webAppRootKey 上下文参数为不同 Web 应用指定不同的属性名：如第一个 Web 应用使用 webapp1.root 而第二个 Web 应用使用 webapp2.root 等，  
这样才不会发生后者覆盖前者的问题。此外，WebAppRootListener 和 Log4jConfigListener 都只能应用在 Web 应用部署后 WAR 文件会解包的 Web 应用服务器上。  
一些 Web 应用服务器不会将 Web 应用的 WAR 文件解包，整个 Web 应用以一个 WAR 包的方式存在（如 Weblogic），此时因为无法指定对应文件系统的 Web 应用根目录，使用这两个监听器将会发生问题。

####特殊字符转义
#####Web 开发者最常面对需要转义的特殊字符类型：  
    * HTML 特殊字符；  
    * JavaScript 特殊字符；  

#####HTML 特殊字符转义  
    * & ：&amp;  
    * " ：&quot;  
    * < ：&lt;  
    * > ：&gt;  

#####JavaScript 特殊字符转义  
    * ' ：/'  
    * " ：/"  
    * / ：//  
    * 走纸换页： /f  
    * 换行：/n  
    * 换栏符：/t  
    * 回车：/r  
    * 回退符：/b  

#####工具类  
{% highlight java %}
JavaScriptUtils.javaScriptEscape(String str);
HtmlUtils.htmlEscape(String str);①转换为HTML转义字符表示
HtmlUtils.htmlEscapeDecimal(String str); ②转换为数据转义表示
HtmlUtils.htmlEscapeHex(String str); ③转换为十六进制数据转义表示
HtmlUtils.htmlUnescape(String str);将经过转义内容还原
{% endhighlight %}

Spring框架下自带了丰富的工具类,在我们开发时可以简化很多工作:  
######1.Resource访问文件资源:  
具体有:	
{% highlight java %}
ResourceUtils.getFile(url);
FileSystemResource(); ClassPathResource();
ServletContextResource(application,url);
{% endhighlight %}

######2.文件操作 FileCopyUtils  
具体有:
{% highlight java %}
FileCopyUtils.copy(Resource.getFile,new File(Resource.getFile(),getParent()+'目标文件名'));
{% endhighlight %}

######3.属性文件操作 PropertiesLoaderUtils    
具体有: 
{% highlight java %}
PropertiesLoaderUtils.loadAllProperties("属性文件名");  --基于类路径
{% endhighlight %}

######4.EncodedResource(Resource对象,"UTF-8") 编码资源(特殊的);   
######5.WebApplicationContextUtils   
######6.StringEscapeutils 编码解码   
