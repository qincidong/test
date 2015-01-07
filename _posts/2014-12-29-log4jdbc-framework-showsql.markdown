---
layout: post
title:  log4jdbc日志框架-显示sql语句
description: 现大家使用的ibatis,hibernate,spring jdbc的sql日志信息，有一点个缺点是占位符与参数是分开打印的,如果想要拷贝sql至PLSQL Developer客户端直接执行，需要自己拼凑sql。而log4jdbc是在jdbc层的一个日志框架，可以将占位符与参数全部合并在一起显示，方便直接拷贝sql在PLSQL Developer等客户端直接执行，加快调试速度。
date:   2014-11-27 16:49:39
categories: blog
tags: log4j sql
---
###一.简单介绍：  
####1.没有使用log4jdbc前sql显示:   
{% highlight sql %}
select username,password from bitth_date > ? and age < ? and username = ?
{% endhighlight %}
####2.使用log4jdbc后sql显示:  
{% highlight sql %}
select username,password from bitth_date > to_date(‘2010-11-11’,’yyyy-mm-dd’) and age < 20 and username = ‘qq2008’ {executed in 2 msec}
{% endhighlight %}
这样就可以直接拷贝上面的sql在PLSQL直接执行. 最后的 {executed in 2 msec} 为SQL执行时间.而如果mysql,日志信息将不会出现 to_date()  
###二.log4jdbc使用：  
####1.spring xml配置（拦截需要处理的dataSource连接）  
{% highlight xml %}
<!-- log4jdbc可以将数据源执行的sql将占位符?替换成字符,并以日志打印出来. log4j配置: log4j.logger.jdbc.sqltiming=INFO    详情请看: http://code.google.com/p/rapid-framework/wiki/log4jdbc
    如oracle示例: 
        原来的sql: select * from user where birth_date = ? and username = ? and age > ?
        转换后sql: select * from user where birth_date = to_date('2010-08-13','yyyy-mm-dd') and username = 'badqiu' and age > 20
     -->
    <bean id="log4jdbcInterceptor" class="net.sf.log4jdbc.DataSourceSpyInterceptor" />
    <bean id="dataSourceLog4jdbcAutoProxyCreator" class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
       <property name="interceptorNames">
           <list>
              <value>log4jdbcInterceptor</value>        
           </list>
       </property>
       <property name="beanNames">
           <list>
              <value>dataSource</value>
           </list>
       </property>
    </bean>
{% endhighlight %}

####2.log4j.properties配置:  
{% highlight java %}
log4j.logger.jdbc.sqlonly=OFF
log4j.logger.jdbc.sqltiming=INFO
log4j.logger.jdbc.audit=OFF
log4j.logger.jdbc.resultset=OFF
log4j.logger.jdbc.connection=OFF
{% endhighlight %}

(日志信息如果全部为off,log4jdbc将不会生效,因此对性能没有任何影响)  
####3.下载slf4j  
>log4jdbc需要依赖slf4j日志框架. [http://www.slf4j.org/](http://www.slf4j.org/)  

###三.扩展说明  
DataSourceSpyInterceptor为我自己扩展的一个拦截器类,扩展主要是使用AOP的方式，因为log4jdbc原来的方式不适合我的项目.具体源码为:  
{% highlight java %}
import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;
public class DataSourceSpyInterceptor implements MethodInterceptor {
private RdbmsSpecifics rdbmsSpecifics = null;
    private RdbmsSpecifics getRdbmsSpecifics(Connection conn) {
        if(rdbmsSpecifics == null) {
            rdbmsSpecifics = DriverSpy.getRdbmsSpecifics(conn);
                }
                return rdbmsSpecifics;
        }
    public Object invoke(MethodInvocation invocation) throws Throwable {
        Object result = invocation.proceed();
        if(SpyLogFactory.getSpyLogDelegator().isJdbcLoggingEnabled()) {
            if(result instanceof Connection) {
                Connection conn = (Connection)result;
                return new ConnectionSpy(conn,getRdbmsSpecifics(conn));
            }
        }
        return result;
    }
}
{% endhighlight %}

###四.相关下载：  
[log4jdbc](http://code.google.com/p/log4jdbc/)  

另外一个对log4jdbc的扩展: [http://code.google.com/p/log4jdbc-remix/](http://code.google.com/p/log4jdbc-remix/) 

转载：[http://blog.sina.com.cn/s/blog_46552dd90100oscb.html](http://blog.sina.com.cn/s/blog_46552dd90100oscb.html) 

