---
layout: post
title:  Spring AOP配置
description: 使用XML和Annotation方式实现Spring AOP
date:   2014-12-29 16:49:39
categories: blog
tags: Spring Aop
---
###一、XML配置方式###
####1.创建一个Service类####

{% highlight java %}
package com.spring.useage.service;

public class UserService {

    public void save() {
        System.out.println("save a user.");
    }

    public void findById(String id) {
        System.out.println("find a user by id");
    }

    public void deleteUser() {
        System.out.println("a user is deleted.");
    }
}
{% endhighlight %}

####2.创建一个切面类####

{% highlight java %}
package com.spring.aop;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.aspectj.lang.JoinPoint;

public class SecurityCheck {
    public final static Log log = LogFactory.getLog(SecurityCheck.class);

    public void check() {
        log.info("权限校验。。。");
    }

    public void addLog(JoinPoint joinPoint) {
        log.info("添加日志。。。");
        String methodName = joinPoint.getSignature().getName();
        log.info("方法名：" + methodName);
        log.info("方法参数：");
        Object[] args = joinPoint.getArgs();
        for (Object obj : args) {
            log.info(obj);
        }
    }
}
{% endhighlight %}        

####3.XML配置的方式如下：####

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:aop="http://www.springframework.org/schema/aop"
               xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
 http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">

            <bean id="securityCheck" class="com.spring.aop.SecurityCheck"></bean>
            <bean id="userService" class="com.spring.useage.service.UserService"></bean>
            <aop:config>
                <aop:aspect id="myAop" ref="securityCheck">
                    <aop:pointcut id="target"
                                  expression="execution(* com.spring.useage.service.*.*(..))" />
                    <aop:before method="check" pointcut-ref="target" />
                    <aop:after method="addLog" pointcut-ref="target" />
                </aop:aspect>
            </aop:config>
        </beans>
{% endhighlight %}        

execution(* com.spring.useage.service.*.*(..))说明：
        第1个*：表示所有返回值类型
        第2个*：表示com.spring.useage.service下的所有类
        第3个*：表示所有方法
        最后的..：表示任意参数
        需要引入的包如下：
        
        ![package](../images/loading.gif)
        
####4.测试类如下：####

{% highlight java %}
package com.spring.aop;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.spring.useage.service.UserService;

public class AopTest {

    /**
     *
     * @author qincd
     * @date Nov 24, 2014 6:30:00 PM
     */
    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        UserService us = ac.getBean(UserService.class);
        us.save();
        us.findById("admin");
        us.deleteUser();
    }

}
{% endhighlight %}   

执行AopTest，输出如下：

2014-11-25 9:59:10 org.springframework.context.support.AbstractApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@14d3343: startup date [Tue Nov 25 09:59:10 CST 2014]; root of context hierarchy
2014-11-25 9:59:10 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [applicationContext.xml]
2014-11-25 9:59:10 org.springframework.beans.factory.support.DefaultListableBeanFactory preInstantiateSingletons
信息: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@1b3f829: defining beans [securityCheck,userService,
org.springframework.aop.config.internalAutoProxyCreator,org.springframework.aop.aspectj.AspectJPointcutAdvisor#0,org.springframework.aop.aspectj.AspectJPointcutAdvisor#1,target];
root of factory hierarchy
2014-11-25 9:59:10 com.spring.aop.SecurityCheck check
信息: 权限校验。。。
save a user.
2014-11-25 9:59:10 com.spring.aop.SecurityCheck addLog
信息: 添加日志。。。
2014-11-25 9:59:10 com.spring.aop.SecurityCheck addLog
信息: 方法名：save
2014-11-25 9:59:10 com.spring.aop.SecurityCheck addLog
信息: 方法参数：
2014-11-25 9:59:10 com.spring.aop.SecurityCheck check
信息: 权限校验。。。
find a user by id
2014-11-25 9:59:10 com.spring.aop.SecurityCheck addLog
信息: 添加日志。。。
2014-11-25 9:59:10 com.spring.aop.SecurityCheck addLog
信息: 方法名：findById
2014-11-25 9:59:10 com.spring.aop.SecurityCheck addLog
信息: 方法参数：
2014-11-25 9:59:10 com.spring.aop.SecurityCheck addLog
信息: admin
2014-11-25 9:59:10 com.spring.aop.SecurityCheck check
信息: 权限校验。。。
a user is deleted.
2014-11-25 9:59:10 com.spring.aop.SecurityCheck addLog
信息: 添加日志。。。
2014-11-25 9:59:10 com.spring.aop.SecurityCheck addLog
信息: 方法名：deleteUser
2014-11-25 9:59:10 com.spring.aop.SecurityCheck addLog
信息: 方法参数：     

###二、Annotation方式###

####1.UserService类加上@Service注解####

{% highlight java %}
 package com.spring.useage.service;

import org.springframework.stereotype.Service;

@Service
public class UserService {

    public void save() {
        System.out.println("save a user.");
    }

    public void findById(String id) {
        System.out.println("find a user by id");
    }

    public void deleteUser() {
        System.out.println("a user is deleted.");
    }
}
{% endhighlight %}

####2.SecurityCheck加上@Aspect和@Component注解####

{% highlight java %}
package com.spring.aop;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class SecurityCheck {
    public final static Log log = LogFactory.getLog(SecurityCheck.class);

    @Before("execution (* com.spring.**.service.*.*(..))")
    public void check() {
        log.info("权限校验。。。");
    }

    @After("execution (* com.spring.**.service.*.*(..))")
    public void addLog(JoinPoint joinPoint) {
        log.info("添加日志。。。");
        String methodName = joinPoint.getSignature().getName();
        log.info("方法名：" + methodName);
        log.info("方法参数：");
        Object[] args = joinPoint.getArgs();
        for (Object obj : args) {
            log.info(obj);
        }
    }
}
{% endhighlight %}

1、@Aspect：意思是这个类为切面类
2、@Componet：因为作为切面类需要Spring管理起来，所以在初始化时就需要将这个类初始化加入Spring的管理；
3、@Befoe：切入点的逻辑(Advice)
4、execution…:切入点语法

applicationContext.xml加入aspectj自动代理，并启用注解。

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
       <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:aop="http://www.springframework.org/schema/aop"
               xmlns:context="http://www.springframework.org/schema/context"
               xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd">

           <aop:aspectj-autoproxy />
           <context:annotation-config />
           <context:component-scan base-package="com.spring" />
       </beans>
{% endhighlight %}

当多个Advice个有相同的织入点。那么我们可以定义一个织入点集合，在需要使用的地方，调用就可以了。
如：

{% highlight java %}
 package com.spring.aop;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class SecurityCheck {
    public final static Log log = LogFactory.getLog(SecurityCheck.class);

    @Before("myPointCut()")
    public void check() {
        log.info("权限校验。。。");
    }

    @After("myPointCut()")
    public void addLog(JoinPoint joinPoint) {
        log.info("添加日志。。。");
        String methodName = joinPoint.getSignature().getName();
        log.info("方法名：" + methodName);
        log.info("方法参数：");
        Object[] args = joinPoint.getArgs();
        for (Object obj : args) {
            log.info(obj);
        }
    }

    @AfterReturning("myPointCut()")
    public void afterReturning() {
        log.info("method after returning...");
    }

    @Around("myPointCut()")
    public void around(ProceedingJoinPoint pjp) throws Throwable {
        log.info("method around start...");
        pjp.proceed();
        log.info("method around end...");
    }

    @AfterThrowing("myPointCut()")
    public void afterThrowing(JoinPoint joinPoint) {
        log.error(joinPoint.getSignature().getName() + "执行异常：" );
    }

    @Pointcut("execution (* com.spring..service.*.*(..))")
    public void myPointCut() {}
}
{% endhighlight %}
        
注意：那个空方法，只是为了给Pointcut起个名字，以方便别处使用<br>

AOP实现动态代理注意

因为Spring要实现AOP(面向切面编程)，需要加入切面逻辑的类就会生成动态代理。在动态代理类中加入切面类从而实现面向切面编程，但生成动态代理存在以下注意事项：
1、 被动态代理的类如果实现了某一个接口，那么Spring就会利用JDK类库生成动态代理。<br>
2、 如果被动态代理的类没有实现某一个接口，那么Spring就会利用CGLIB类库直接修改二进制码来生成动态代理(因为利用JDK生成动态代理的类必须实现一个接口)，需要在项目中引用CGLIB类库
参考：[http://blog.csdn.net/yuqinying112/article/details/7335416](http://blog.csdn.net/yuqinying112/article/details/7335416)   
        
Jekyll相关参考：[jekyll][jekyll]，[jekyll-help][jekyll-help]
[jekyll]:      http://jekyllrb.com
[jekyll-help]: https://github.com/jekyll/jekyll-help

Markdown语法参考：[Markdown语法参考](http://wowubuntu.com/markdown/)
