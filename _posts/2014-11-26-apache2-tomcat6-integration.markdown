---
layout: post
title: Apache2 + Tomcat6集群
description: Apache2 + Tomcat6集群
date:   2014-11-26 16:49:39
categories: blog
tags: apache tomcat 集群
---
参考：[http://wiki.bsdn.org/pages/viewpage.action?pageId=2621465][1]  [http://developer.51cto.com/art/201102/246338.htm][2]
[1]:http://wiki.bsdn.org/pages/viewpage.action?pageId=2621465
[2]:http://developer.51cto.com/art/201102/246338.htm

###一、软件准备  
Apache 2.2 : [http://httpd.apache.org/download.cgi](http://httpd.apache.org/download.cgi)，下载msi安装程序，选择no ssl版本  
Tomcat 6.0 : [http://tomcat.apache.org/download-60.cgi](http://tomcat.apache.org/download-60.cgi)，下载Tomcat 6.0.18 zip文件       

###二、软件安装  
把Apache安装为运行在80端口的Windows服务，安装成功后在系统服务列表中可以看到Apache2.2服务。  
对于已安装IIS的机器，在启动Apache服务之前必须首先停止IIS Admin服务，不然会因为端口冲突而无法启动。  
服务启动后在浏览器中输入http://localhost进行测试，如果能看到一个"It works!"的页面就代表Apache已经正常工作了。  

解压tomcat zip文件到两个文件夹，分别为t1和t2，以下均以t1和t2代表两个tomcat服务器。  
配置JAVA_HOME和CLASSPATH系统环境变量，分别启动t1和t2，确保tomcat可用，然后关闭tomcat。  
本文仅为讲解配置过程，Apache和tomcat均工作在同一台机器上。 
实际部署时没有任何限制，Apache和单个tomcat可以分别部署在不同的服务器上。  

###三、Apache配置  
Apache 2.2集成了mod_jk功能，相对于1.3版本，不需要再进行繁琐的worker.properties配置，配置过程大幅简化。  
首先，在Apache安装目录下找到conf/httpd.conf文件，以文本编辑器打开。  
去掉以下文本前的注释符(#)以便让Apache在启动时自动加载代理(proxy)模块。  
{% highlight xml %}
LoadModule proxy_module modules/mod_proxy.so    
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so    
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so    
LoadModule proxy_connect_module modules/mod_proxy_connect.so    
LoadModule proxy_ftp_module modules/mod_proxy_ftp.so    
LoadModule proxy_http_module modules/mod_proxy_http.so   
{% endhighlight %}
向下拉动文档找到节点，在DirectoryIndex index.html后加上index.jsp，这一步只是为了待会配置完tomcat后能看到小猫首页，可以不做。  
继续下拉文档找到Include conf/extra/httpd-vhosts.conf，去掉前面的注释符。  
用文本编辑器打开conf/extra/httpd-vhosts.conf，配置虚拟站点，在最下面加上  
{% highlight xml %}
<VirtualHost *:80>   
    ServerAdmin 管理员邮箱    
    ServerName 域名（没有可用IP地址代替）    
    ServerAlias localhost   
    ProxyPass / balancer://cluster/ stickysession=jsessionid nofailover=On   
    ProxyPassReverse / balancer://cluster/   
    ErrorLog "logs/lbtest-error.log"  
    CustomLog "logs/lbtest-access.log" common  
</VirtualHost> 
{% endhighlight %}
这里balancer://是告诉Apache需要进行负载均衡的代理，后面的cluster是集群名，可以随意取，两个日志引擎ErrorLog负责记录错误，  
CustomLog负责记录所有的http访问以及返回状态，日志名可以自己取，笔者取为lbtest。  
httpd-vhosts.conf配置完毕，回到httpd.conf，在文档最下面加上  
{% highlight xml %}
ProxyRequests Off   
<proxy balancer://cluster>   
  BalancerMember ajp://127.0.0.1:8009 loadfactor=1 route=node1 
  BalancerMember ajp://127.0.0.1:9009 loadfactor=1 route=node2 
</proxy> 
{% endhighlight %}	 
ProxyRequests Off 是告诉Apache需要使用反向代理(利用Apache进行负载均衡必须使用反向代理，  
关于更多负载均衡和反向代理详情可以参阅笔者另一篇博客http://zyycaesar.javaeye.com/admin/blogs/293839)，   
用于配置工作在tomcat集群中的所有节点，这里的"cluster"必须与上面的集群名保持一致。  
Apache通过ajp协议与tomcat进行通信，ip地址和端口唯一确定了tomcat节点和配置的ajp接受端口。  
loadfactor是负载因子，Apache会按负载因子的比例向后端tomcat节点转发请求，负载因子越大，对应的tomcat服务器就会处理越多的请求，  
如两个tomcat都是1，Apache就按1：1的比例转发，如果是2和1就按2：1的比例转发。  
route参数对应后续tomcat配置中的引擎路径(jvmRoute)。  

重启Apache服务，如果此时访问http://localhost/将会返回503错误，打开刚才配置的错误日志logs/lbtest-error.log，  
可以看到错误原因是因为后台服务器没有响应，因为此时tomcat尚未配置和启动。  

###四、Tomcat配置  
如果仅仅为了配置一个可用的集群，Tomcat的配置将会非常简单。  
分别打开t1和t2的server.xml配置文件，对于t1，尽量采用默认的设置，而对t2作较大改动以避免与t1冲突。  
如果t2和t1不在同一台服务器上运行，对于端口就不需做改动。  

首先是配置关闭端口，找到，t1不变，把t2改为9005。
{% highlight xml %}  
<Server port="9005" shutdown="SHUTDOWN">  
{% endhighlight %}	 

下面配置Connector的端口，找到non-SSL HTTP/1.1 Connector，即tomcat单独工作时的默认Connector，  
保留t1默认配置，在8080端口侦听，而把t2设置为在9090端口侦听。  
{% highlight xml %}  
<Connector port="9090" protocol="HTTP/1.1" 
               connectionTimeout="20000" 
               redirectPort="9443" />
{% endhighlight %}	 

往下找到AJP 1.3 Connector，，这是tomcat接收从Apache过来的ajp连接请求时使用的端口，保留t1默认设置，把t2端口改为9009。  
注意，这里的端口对应Apache httpd.conf中BalancerMember中配置的ajp连接端口。  
{% highlight xml %}  
<!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector port="9009" protocol="AJP/1.3" redirectPort="9443" />
{% endhighlight %}	 

继续向下配置引擎，在Engine标签中加入属性jvmRoute="node2"（t1中为 jvmRoute="node1"）  

####一、第1种session复制方式  
=================================================================================================================  
将t1,t2<Engine>标签中的<cluster>标签注释打开就行了。  
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓  
1).这是默认的session复制方式，即集群的session复制默认是DeltaManager，是all to all的复制，意思是将集群下1个tomcat应用下的session  
对所有的集群中的节点进行复制，即使那个节点没有发布应用。显然是不好的方式，但这在小规模的集群下并没神马问题。  
=================================================================================================================    
     
####二、第2种session复制方式   
=================================================================================================================   
在<Engine>标签中加入如下代码：  
{% highlight xml %}  
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
                 channelSendOptions="6">
          <Manager className="org.apache.catalina.ha.session.BackupManager"
                   expireSessionsOnShutdown="false"
                   notifyListenersOnReplication="true"
                   mapSendOptions="6"/>
          <!--
          <Manager className="org.apache.catalina.ha.session.DeltaManager"
                   expireSessionsOnShutdown="false"
                   notifyListenersOnReplication="true"/>
          -->
          <Channel className="org.apache.catalina.tribes.group.GroupChannel">
            <Membership className="org.apache.catalina.tribes.membership.McastService"
                        address="228.0.0.4"
                        port="45564"
                        frequency="500"
                        dropTime="3000"/>
            <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                      address="auto"
                      port="5000"
                      selectorTimeout="100"
                      maxThreads="6"/>
            <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
              <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
            </Sender>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.ThroughputInterceptor"/>
          </Channel>
          <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
                 filter=".*\.gif|.*\.js|.*\.jpeg|.*\.jpg|.*\.png|.*\.htm|.*\.html|.*\.css|.*\.txt"/>
          <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
                    tempDir="/tmp/war-temp/"
                    deployDir="/tmp/war-deploy/"
                    watchDir="/tmp/war-listen/"
                    watchEnabled="false"/>
          <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
</Cluster>
{% endhighlight %}	 
对于t2，修改Receiver的port为4000 ~ 4100之间的数，并且与t1不重复。  
↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓  
这种session复制方式是采用BackupManager，这种适合于大规模的集群。这种只有session中的值发生改变的时候才会复制session。  
=================================================================================================================  
如果不是大规模的集群， 选择第一种方式就行了。  
如果要创建多组Cluster，只需要修改Receiver的address和port就行了。  
这里的jvmRoute对应Apache httpd.conf中BalancerMember中配置的route参数。   
这里的配置是为了可以在集群中的所有tomcat节点间共享会话(Session)。  

server.xml的配置修改完毕，下一步需要对具体的应用进行配置。  
在webapps目录下新建test目录，在test目录下新建test.jsp文件，代码如下：  
{% highlight xml %} 
<%@ page contentType="text/html; charset=GBK" %> 
<%@ page import="java.util.*" %> 
<html><head><title>Cluster App Test</title></head> 
<body> 
Server Info:  
<%  
out.println(request.getLocalAddr() + " : " + request.getLocalPort()+"<br>");%> 
<%  
  out.println("<br> ID " + session.getId()+"<br>");   
  String dataName = request.getParameter("dataName");  
  if (dataName != null && dataName.length() > 0) {  
     String dataValue = request.getParameter("dataValue");  
     session.setAttribute(dataName, dataValue);  
  }    
  out.print("<b>Session 列表</b>");    
  Enumeration e = session.getAttributeNames();  
  while (e.hasMoreElements()) {  
     String name = (String)e.nextElement();  
     String value = session.getAttribute(name).toString();  
     out.println( name + " = " + value+"<br>");  
         System.out.println( name + " = " + value);  
   }  
%> 
  <form action="test.jsp" method="POST"> 
    名称:<input type=text size=20 name="dataName"> 
     <br> 
    值:<input type=text size=20 name="dataValue"> 
     <br> 
    <input type=submit> 
   </form> 
</body> 
</html> 
{% endhighlight %}	 
在test目录下继续新建WEB-INF目录和web.xml，在节点下加入<distributable/>，  
这一步非常重要，是为了通知tomcat服务器，当前应用需要在集群中的所有节点间实现Session共享。  
如果tomcat中的所有应用都需要Session共享，也可以把conf/context.xml中的改为，这样就不需对所有应用的web.xml再进行单独配置。

启动t1，待t1启动完成后再启动t2。再次访问http://localhost，可以看到小猫页面。  
访问http://localhost/test/test.jsp。可以看到包括服务器地址，端口，sessionid等信息在内的页面。

注意这里的sessionid，与平常的sessionid相比多了小数点和后面的部分，这里的node1即处理当前请求tomcat服务器的jvmRoute，  
通过这里可以知道是集群中的哪一个服务器处理了当前请求。在文本框中输入名称和值，点击按钮，信息就保存到了Session中，  
并且显示到页面上。不断点击按钮，可以发现输入的信息并未丢失，而且sessionid小数点之前的部分保持不变，  
而小数点后面的字符不停的变化，表明是由不同的tomcat服务器处理了这些请求。  
这样就实现了负载均衡，并且集群中的不同节点间可以实现会话的共享。  
此时如果停止一个tomcat服务器t2，Apache将会自动把后续请求转发到集群中的其他服务器即t1。  
重启t2后，Apache会自动侦测到t2的状态为可用，然后会继续在t1和t2间进行负载均衡。  

如果需要向集群中增加节点，首先需要对tomcat作类似配置，然后修改Apache httpd.conf，增加BalancerMember，指向新增的tomcat即可。  