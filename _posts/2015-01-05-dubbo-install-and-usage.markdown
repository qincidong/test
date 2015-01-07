---
layout: post
title:  dubbo的安装和使用
description: dubbo的安装和使用
date:   2015-01-05 16:49:39
categories: blog
tags: dubbo
---
##背景
(#)

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进。
![img](http://alibaba.github.io/dubbo-doc-static/dubbo-architecture-roadmap.jpg-version=1&modificationDate=1331143666000.jpg)
**单一应用架构**  
当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。  
此时，用于简化增删改查工作量的 数据访问框架(ORM) 是关键。

**垂直应用架构**  
当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。  
此时，用于加速前端页面开发的 Web框架(MVC) 是关键。

**分布式服务架构**  
当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。  
此时，用于提高业务复用及整合的 分布式服务框架(RPC) 是关键。

**流动计算架构**  
当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。  
此时，用于提高机器利用率的 资源调度和治理中心(SOA) 是关键。


##需求
(#)
![img](http://alibaba.github.io/dubbo-doc-static/dubbo-service-governance.jpg-version=1&modificationDate=1331887614000.jpg)
在大规模服务化之前，应用可能只是通过RMI或Hessian等工具，简单的暴露和引用远程服务，通过配置服务的URL地址进行调用，通过F5等硬件进行负载均衡。

**(1) 当服务越来越多时，服务URL配置管理变得非常困难，F5硬件负载均衡器的单点压力也越来越大。**

此时需要一个服务注册中心，动态的注册和发现服务，使服务的位置透明。

并通过在消费方获取服务提供方地址列表，实现软负载均衡和Failover，降低对F5硬件负载均衡器的依赖，也能减少部分成本。

**(2) 当进一步发展，服务间依赖关系变得错踪复杂，甚至分不清哪个应用要在哪个应用之前启动，架构师都不能完整的描述应用的架构关系。**

这时，需要自动画出应用间的依赖关系图，以帮助架构师理清理关系。

**(3) 接着，服务的调用量越来越大，服务的容量问题就暴露出来，这个服务需要多少机器支撑？什么时候该加机器？**

为了解决这些问题，第一步，要将服务现在每天的调用量，响应时间，都统计出来，作为容量规划的参考指标。

其次，要可以动态调整权重，在线上，将某台机器的权重一直加大，并在加大的过程中记录响应时间的变化，直到响应时间到达阀值，记录此时的访问量，再以此访问量乘以机器数反推总容量。

以上是Dubbo最基本的几个需求，更多服务治理问题参见：

[service-governance-process](http://code.alibabatech.com/blog/experience_1402/service-governance-process.html)

##架构
(#)
![img](http://alibaba.github.io/dubbo-doc-static/dubbo-architecture.jpg-version=1&modificationDate=1330892870000.jpg)
节点角色说明：  
Provider: 暴露服务的服务提供方。  
Consumer: 调用远程服务的服务消费方。  
Registry: 服务注册与发现的注册中心。  
Monitor: 统计服务的调用次调和调用时间的监控中心。  
Container: 服务运行容器。  

调用关系说明：  
0. 服务容器负责启动，加载，运行服务提供者。  
1. 服务提供者在启动时，向注册中心注册自己提供的服务。  
2. 服务消费者在启动时，向注册中心订阅自己所需的服务。  
3. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。  
4. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。  
5. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

####(1) 连通性：
注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小  
监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示  
服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销  
服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销  
注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外  
注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者  
注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表  
注册中心和监控中心都是可选的，服务消费者可以直连服务提供者  

####(2) 健状性：
监控中心宕掉不影响使用，只是丢失部分采样数据  
数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务  
注册中心对等集群，任意一台宕掉后，将自动切换到另一台  
注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯  
服务提供者无状态，任意一台宕掉后，不影响使用  
服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

####(3) 伸缩性：
注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心  
服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者  

####(4) 升级性：
当服务集群规模进一步扩大，带动IT治理结构进一步升级，需要实现动态部署，进行流动计算，现有分布式服务架构不会带来阻力：
![img](http://alibaba.github.io/dubbo-doc-static/dubbo-architecture-future.jpg-version=1&modificationDate=1329978098000.jpg)
Deployer: 自动部署服务的本地代理。  
Repository: 仓库用于存储服务应用发布包。  
Scheduler: 调度中心基于访问压力自动增减服务提供者。  
Admin: 统一管理控制台。

##安装
###一、本地服务
#####1、定义服务接口: (该接口需单独打包，在服务提供方和消费方共享)
{% highlight java %}
public interface CustomerService {  
    public String getName();  
}  
{% endhighlight %}

#####2、在服务提供方实现接口：(对服务消费方隐藏实现)
{% highlight java %}
public class CustomerServiceImpl implements CustomerService{  
    @Override  
    public String getName() {  
        System.out.print("我打印");  
        return "打印结果";  
    }  
}  
{% endhighlight %}

#####3、然后引入dubbo的几个包
dubbo-2.5.3.jar  
log4j.jar  
netty-3.5.7.Final.jar  
slf4j.jar  
slf4j-log4j.jar  
zkclient.jar  
zookeeper.jar

注：如果使用的Maven，可以直接加入下面的依赖，Maven会自动下载需要的包：
{% highlight xml %}
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.5.3</version>
</dependency>
{% endhighlight %}        
是下面几个文件：
![img](http://uploadingit.com/file/g6o5pbahoujer7ec/QQ%E6%88%AA%E5%9B%BE20150105145834.png)
这个是使用的multicast，参考：[http://alibaba.github.io/dubbo-doc-static/User+Guide-zh.htm](http://alibaba.github.io/dubbo-doc-static/User+Guide-zh.htm)

如果是zookeeper形式，添加下面的依赖：
{% highlight xml %}
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>2.5.3</version>
    </dependency>

    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.5.0-alpha</version>
    </dependency>
    <dependency>
        <groupId>com.github.sgroschupf</groupId>
        <artifactId>zkclient</artifactId>
        <version>0.1</version>
    </dependency>
</dependencies>
{% endhighlight %}    

#####4、用Spring配置声明暴露服务：
新建applicationProvider.xml，配置内容如下：
{% highlight xml %}   
<?xml version="1.0" encoding="UTF-8"?>    
<beans xmlns="http://www.springframework.org/schema/beans"    
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"    
    xsi:schemaLocation="http://www.springframework.org/schema/beans    
        http://www.springframework.org/schema/beans/spring-beans.xsd    
        http://code.alibabatech.com/schema/dubbo    
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd ">     
    <!-- 具体的实现bean -->    
    <bean id="demoService" class="com.jinbin.service.customer.CustomerServiceImpl" />    
    <!-- 提供方应用信息，用于计算依赖关系 -->    
    <dubbo:application name="xixi_provider"  />      
    <!-- 使用multicast广播注册中心暴露服务地址     
    <dubbo:registry address="multicast://localhost:1234" />-->     
    <!-- 使用zookeeper注册中心暴露服务地址 -->    
    <dubbo:registry address="zookeeper://192.168.1.3:2181" />       
    <!-- 用dubbo协议在20880端口暴露服务 -->    
    <dubbo:protocol name="dubbo" port="20880" />    
    <!-- 声明需要暴露的服务接口 -->    
    <dubbo:service interface="com.jinbin.service.customer.CustomerService" ref="demoService" />    
</beans>    
{% endhighlight %}

我这里暴露服务器的地址交由zookeeper来管理的，使用者首先先要安装zookeeper应用才能使用此功能，相关安装步骤请参看相关博文
 
5、加载Spring配置，并调用远程服务：(也可以使用IoC注入)
{% highlight java %}
public class DubooProvider {  
    public static void main(String[] args) {  
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(    
                new String[]{"applicationProvider.xml"});    
        context.start();    
        System.out.println("Press any key to exit.");    
        try {  
            System.in.read();  
        } catch (IOException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }    
    }  
}  
{% endhighlight %}

并且启动，使其进入启动状态。

以上为服务器提供者的完整步骤，功能接口都已经写好，下面我们就开始怎么远程调用

###二、服务消费者

#####1、新建个配置文件applicationConsumer.xml，内容如下：
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>    
<beans xmlns="http://www.springframework.org/schema/beans"    
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"    
    xsi:schemaLocation="http://www.springframework.org/schema/beans    
        http://www.springframework.org/schema/beans/spring-beans.xsd    
        http://code.alibabatech.com/schema/dubbo    
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd ">          
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->    
    <dubbo:application name="consumer-of-helloworld-app" />       
      <!-- 使用multicast广播注册中心暴露发现服务地址 -->    
    <dubbo:registry  protocol="zookeeper" address="zookeeper://192.168.1.3:2181" />         
      <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->    
    <dubbo:reference id="demoService" interface="com.jinbin.service.customer.CustomerService" />    
</beans>    
{% endhighlight %}

为了在web中使用，我们在web.xml中配置在spring启动读取过程中
{% highlight xml %}
<context-param>  
    <param-name>contextConfigLocation</param-name>  
    <param-value>/WEB-INF/application.xml /WEB-INF/applicationConsumer.xml</param-value>  
</context-param>  
{% endhighlight %}

#####2、接口调用
调用过程很简单，先把接口文件打成jar包，然后在此工程中进行引用

在springmvc调用程序如下：
{% highlight java %}
@Autowired CustomerService demoService ;       
{% endhighlight %}

{% highlight java %}
@RequestMapping(value="duboo1")  
public void duboo1(){  
   demoService.getName();  
}
{% endhighlight %}

即可执行成功

###三、dubbo-admin的使用
下载dubbo-admin-2.5.3.war  
将其放到tomcat下面，配置dubbo.properties，
{% highlight xml %}
vi
 webapps/ROOT/WEB-INF/dubbo.properties
dubbo.properties
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.admin.root.password=root
dubbo.admin.guest.password=guest
{% endhighlight %}

修改zookeeper的URL和端口
     
启动:  
./bin/startup.sh  
打开，直接访问首页如下：
![img](http://alibaba.github.io/dubbo-doc-static/dubbo-search.png-version=1&modificationDate=1343130241000.png)
![img](http://alibaba.github.io/dubbo-doc-static/dubbo-providers.png-version=1&modificationDate=1343130241000.png)
![img](http://alibaba.github.io/dubbo-doc-static/dubbo-consumers.png-version=1&modificationDate=1343130241000.png)
![img](http://alibaba.github.io/dubbo-doc-static/dubbo-applications.png-version=1&modificationDate=1343130241000.png)
添加路由规则页面
![img](http://alibaba.github.io/dubbo-doc-static/dubbo-add-route.png-version=1&modificationDate=1343130241000.png)
添加动态配置页面
![img](http://alibaba.github.io/dubbo-doc-static/dubbo-add-config.png-version=1&modificationDate=1343130245000.png)

转载：[http://blog.csdn.net/songjinbin/article/details/26006621](http://blog.csdn.net/songjinbin/article/details/26006621)