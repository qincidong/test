---
layout: post
title:  Maven依赖之Scope
description: Maven依赖之Scope
date:   2013-12-01 16:49:39
categories: blog
tags: maven
---
我们知道，可以通过dependency标签来添加依赖，一般情况下我们也只是需要关注groupId,artifaceId和version。   
但是，我们也知道，并不是所有的JAR都要在编译，测试，运行，打包……各个阶段都存在，比如junit.jar。  
在打包成WAR的时候是没必要将单元测试代码也打进去的，junit.jar也没必要打进去。  
在Maven中的dependency标签中提供了scope属性，它包含5个值：  
compile:默认值，所以在编译，测试，运行，发布各个阶段都存在。  
test:只在编译和运行测试代码时使用，不会打到WAR包里面。  
provided:类似compile，它不会在打包时打进去，比如servlet-api.jar，它希望容器、使用者等来提供这个依赖。  
runtime:只在运行时使用，比如JDBC的驱动包。  
system:与provided很像，不过maven不会从仓库中查找，需要你自己显示的指定systemPath。这个不常用。

比如：
{% highlight xml %}
<dependency>
     <groupId>log4j</groupId>
     <artifactId>log4j</artifactId>
     <version>1.2.17</version>
     <scope>system</scope>
     <systemPath>F:\log4j-1.2.17.jar</systemPath>
</dependency>
{% endhighlight %}

这样你自己指定了JAR的位置之后，Maven就不会从仓库去查找依赖了。

依赖具有传递性。比如B依赖A，Maven会把A依赖的文件添加到B的Maven依赖中。不过，只有scope是compile的才会传递过去。

假设下面的情形：
模块A中用了dbunit，dbunit是依赖junit3.8.2的。这个看Maven的依赖继承树就知道。如图：
![img](http://images.cnitblog.com/blog/548748/201312/01171808-31c2f00db2224dd9bcd02ac29c45a2d5.png)
maven默认添加了Junit3.8.1的依赖，但是dbunit依赖了3.8.2，如上图，因为跟3.8.1产生冲突，所以junit3.8.2没有引入。

在模块B中我使用了junit4.10做单元测试。  
在模块C中依赖A和B。那么传递到C中的juni版本是多少？（假定C中先添加对A的依赖）。  
实际情况是：添加的是对junit3.8.1的依赖。

如果C中先添加对B的依赖呢？（B中对junit的依赖的是4.10）  
实际情况是：C中添加的是对junit4.10的依赖。

假定C依赖A，不再依赖B了。C中使用的是junit4.10，A中仍然使用的是dbunit，它依赖的是3.8.2。  
那么，在C中添加对A的依赖后（假定C中没有显示添加对junit的依赖），那么实际就会引入3.8.1的。  
但是C中需要的是junit4.10，如何解决这个问题呢？  
可以使用exclusion标签来排除依赖。如：
{% highlight xml %}
<dependency>
    <groupId>com.maven.dependency.test.a</groupId>
    <artifactId>A</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <exclusions>
        <exclusion>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </exclusion>
    </exclusions>
</dependency>
{% endhighlight %}

这样A就不会添加junit的依赖到C中了。这个例子举的其实不太合适，因为你如果在C中指定了Junit版本是4.10，那么是不会见A中的3.8.1传递过来的。  
其实想说就是如果遇到JAR版本冲突问题，可通过exclusions来解决。