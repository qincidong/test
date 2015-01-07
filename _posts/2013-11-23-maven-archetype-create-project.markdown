---
layout: post
title:  使用maven archetype创建Maven项目
description: 使用maven archetype创建Maven项目
date:   2013-11-23 16:49:39
categories: blog
tags: maven
---
在前面说过了Maven项目的结构式固定的，因此maven也提供了一个命令来创建这种情况的项目。  
现在使用maven archetype命令在workspace目录下创建maven_ch03项目。  
打开cmd，切换到F:/study/workspace，输入mvn archetype:generate命令，回车。
![img](http://images.cnitblog.com/blog/548748/201311/23185254-aa510d0726764d1a87971fc17d2ef994.png)
列出了879种创建方式，默认是328这个：  
328: remote -> org.apache.maven.archetypes:maven-archetype-quickstart (An archetype which contains a  
 sample Maven project.)  
Ok，就选择这个。
![img](http://images.cnitblog.com/blog/548748/201311/23185444-f4863365e5a44f56af03bee394586f55.png)
1-4都是内部测试版本，选择版本最高的1.1，输入6，如上图。
![img](http://images.cnitblog.com/blog/548748/201311/23185947-053f0ea167a34fec8be976fb8e560bfc.png)
如上图，然后就是输入groupId,artifactId,version,package。有一些项是有默认值的，如果你不需要修改回车即可。  
Ok，如上图，项目已经创建成功了。
![img](http://images.cnitblog.com/blog/548748/201311/23190142-e092698413e1478495810afc00f9e9a0.png)
如上图，到F:study/workspace，maven已经自动创建了项目maven_ch03.  
Maven在src/main/java下默认创建了App.java，代码如下：
{% highlight java %}
package com.purple_river.itat.mvn;
 
/**
 * Hello world!
 *
 */
public class App
{
    public static void main( String[] args )
    {
        System.out.println( "Hello World!" );
    }
}
{% endhighlight %}

在测试目录中生成了AppTest.java，代码如下：
{% highlight java %}
package com.purple_river.itat.mvn;
 
import junit.framework.Test;
import junit.framework.TestCase;
import junit.framework.TestSuite;
 
/**
 * Unit test for simple App.
 */
public class AppTest
    extends TestCase
{
    /**
     * Create the test case
     *
     * @param testName name of the test case
     */
    public AppTest( String testName )
    {
        super( testName );
    }
 
    /**
     * @return the suite of tests being tested
     */
    public static Test suite()
    {
        return new TestSuite( AppTest.class );
    }
 
    /**
     * Rigourous Test :-)
     */
    public void testApp()
    {
        assertTrue( true );
    }
}
{% endhighlight %}

那么，再看看pom.xml吧：
{% highlight xml %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.purple_river.itat.mvn</groupId>
  <artifactId>maven_ch03</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>maven_ch03</name>
  <url>http://maven.apache.org</url>
 
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
{% endhighlight %}

可以看到Maven已经帮我们定义了junit的依赖，版本是3.8.1。  
不过，一般是通过下面的方式来创建：  
F:\study\workspace>mvn archetype:generate -DgroupId=com.purple_river.itat.mvn -DartifactId=maven_ch0
3 -Dversion=0.0.1SNAPSHOT。
可以通过-D参数名=参数值来指定。  
Ok，到这里，使用maven archetype创建Maven项目就结束了。
