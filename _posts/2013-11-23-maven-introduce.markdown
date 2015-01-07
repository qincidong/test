---
layout: post
title:  Maven的初步使用
description: Maven的初步使用
date:   2013-11-23 16:49:39
categories: blog
tags: maven
---
这里通过一个例子来说明如何使用Maven。  
我的工作空间是F:/study/workspace。  
在工作空间下新建工程maven_ch01，目录结构如下：  
maven_ch01  
-------------src  
-----------------main  
------------------------java  
-----------------test  
------------------------java  
-------------pom.xml  
这种结构式Maven推荐的工程结构。  
在maven_ch01/src/main/java下新建Java文件HelloMaven.java，文件在包com.purple_river.itat.maven下。  
就一个简单的sayHello方法，如下：
{% highlight java %}
package com.purple_river.itat.maven;
 
public class HelloMaven {
    public String sayHello(String name) {
        return "Hello:" + name;
    }
}
{% endhighlight %}

在src/test/java下建立与src/main/java下相同的包com.purple_river.iata.maven，并新建TestHelloMaven.java。

代码如下：
{% highlight java %}
package com.purple_river.itat.maven;
 
import org.junit.*;
import static org.junit.Assert.*;
 
public class TestHelloMaven {
     
    @Test
    public void testSayHello() {
        HelloMaven hm = new HelloMaven();
        String result = hm.sayHello("maven");
        assertEquals(result,"Hello:maven");
    }
}
{% endhighlight %}

如你所见，这个测试类中有一个测试方法，就是测试我们在src/main/java下新建的HelloMaven类的sayHello方法。
可能你也发现了，测试类TestHelloMaven是需要junit.jar的，但是我们的工程中病没有，那么我们怎么运行呢？

刚刚忘了说pom.xml文件的内容，pom.xml文件的内容如下：
{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="htp://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.purple_river.itat.maven</groupId>
    <artifactId>maven_ch01</artifactId>
    <version>0.0.1-SWAPSHOT</version>
     
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId> 
            <version>4.8</version>
        </dependency>
    </dependencies>
</project>   
{% endhighlight %}

pom.xml文件基本节点介绍：  
&lt;project&gt;:文件的根节点.  
&lt;modelversion&gt;:pom.xml使用的对象模型版本.  
&lt;groupId&gt;:创建项目的组织或团体的唯一Id.  
&lt;artifactId&gt;:项目的唯一Id,可视为项目名.  
&lt;packaging&gt;:打包物的扩展名，一般有JAR,WAR,EAR等  
&lt;version&gt;:产品的版本号.  
&lt;name&gt;:项目的显示名，常用于Maven生成的文档。  
&lt;url&gt;:组织的站点，常用于Maven生成的文档。  
&lt;description&gt;:项目的描述，常用于Maven生成的文档。
&lt;dependencies&gt;节点定义工程中的依赖，一个<dependency>标签定义一个依赖，比如上面的<dependecies>节点定义我们工程使用了junit，使用的junit版本是4.8。

下面使用mvn compile命令来编译java源文件：  
1.在cmd中将位置切换到F:\study\workspace\maven_ch01>；  
2.输入mvn compile命令；
![img](http://images.cnitblog.com/blog/548748/201311/23161111-8d5600a178504d0cb7db2a75c0bbbf9d.png)
看到没有？maven会自动下载它所需的依赖文件，最后看到BUILD SUCCESS。

回到工程maven_ch01下，会发现生成了一个target目录。如下：
![img](http://images.cnitblog.com/blog/548748/201311/23161302-5547e2124f074fb289326e84378c4d38.png)
可以看到已经变异了HelloMaven类。

到我们自定义的本地仓库看看，Maven所需的文件都下载在这里，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23161428-75ec05b4493045efb2e974150bdc52d0.png)
我们工程所需的junit也下载在这里
![img](http://images.cnitblog.com/blog/548748/201311/23161529-66806137ad1f4ef9876d9b5ea6766550.png)
Ok，咱们在编译一次看看Maven还会不会从网络下载所依赖的文件呢
![img](http://images.cnitblog.com/blog/548748/201311/23161654-40a74e52706b4767b58e66ab1da053ac.png)
看到没有，这次Maven并没有从网络下载依赖的文件了吧！只有在第一次编译的时候才会下载它依赖的文件，下载到本地后再次运行就会使用本地仓库中的依赖文件。

使用mvn test命令还运行单元测试。如图：
![img](http://images.cnitblog.com/blog/548748/201311/23164416-6a15799ff3ce462092bcfa95a3c70e6e.png)
此时target目录中多生成了3个目录，如下图：
![img](http://images.cnitblog.com/blog/548748/201311/23164601-3d761612f4454978b674d36530916825.png)
test-classes：存放编译的测试类

surefire-reports：存放测试报告。包括成功的和失败的详细信息。

将测试类修改一下：
{% highlight java %}
package com.purple_river.itat.maven;
 
import org.junit.*;
import static org.junit.Assert.*;
 
public class TestHelloMaven {
     
    @Test
    public void testSayHello() {
        HelloMaven hm = new HelloMaven();
        String result = hm.sayHello("maven");
        assertEquals(result,"Hello:maven1");
    }
}
{% endhighlight %}

将assertEquals(result,"Hello:maven")改成了assertEquals(result,"Hello:maven1")，那么运行测试是会失败的。

重新只需mvn test命令来执行测试：
![img](http://images.cnitblog.com/blog/548748/201311/23165045-3aa40c8b20d84ba484eb7fd1ed0acdc2.png)
首先呢，命令行中显示运行的结果是失败的。我们到测试报告中看看。
![img](http://images.cnitblog.com/blog/548748/201311/23165216-6df64dae54394d76b0a420e5d36f4161.png)
看到没有，maven记录了详细的错误信息。

 
使用mvn package命令打包。

如图：
![img](http://images.cnitblog.com/blog/548748/201311/23165656-48c9606ffa4548f1802a0eecd5c9f19c.png)
在target目录中生成了maven_ch01_0.0.1_SWAPSHOT.jar，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23165810-af74312967ee400fb099f390975e4086.png)
在实际开发中，可能是分模块开发，那每个模块之间的依赖呢？

新建工程maven_ch02，与maven_ch01下有一个HelloMaven02.java文件，

代码如下：
{% highlight java %}
package com.purple_river.itat.maven;
 
public class HelloMaven02 {
    public static void main(String[] args) {
        HelloMaven hm = new HelloMaven();
        String str = hm.sayHello("maven");
        System.out.println(str);
    }
}
{% endhighlight %}

这个文件中要使用maven_ch01中的HelloMaven类，那怎么使用maven来管理依赖呢？

maven_ch02下的pom.xml文件如下：
{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="htp://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.purple_river.itat.maven</groupId>
    <artifactId>maven_ch02</artifactId>
    <version>0.0.1-SWAPSHOT</version>
     
    <dependencies>
        <dependency>
            <groupId>com.purple_river.itat.maven</groupId>
            <artifactId>maven_ch01</artifactId>
            <version>0.0.1-SWAPSHOT</version>
        </dependency>
    </dependencies>
</project>   
{% endhighlight %}

关键看<dependencies/>标签，这里定义的就是对maven_ch01的依赖。groupId,artifactId,version都是在maven_ch01中的pom.xml中定义的。

在cmd中切换到maven_ch02，执行mvn compile命令，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23170735-b03b07d0423b4a01bbe1588b37aff532.png)
看到上面的Downloading:http://repo.maven.apache.org/maven2/com/purple_river/itat/maven/maven_ch01/0.0.1_SWAPSHOT/maven_ch01-0.0.1-SWAPSHOT.pom和Downloading:http://repo.maven.apache.org/maven2/com/purple_river/itat/maven/maven_ch01/0.0.1_SWAPSHOT/maven_ch01-0.0.1-SWAPSHOT.jar这2行了吗?因为我们在maven_ch02的pom.xml中定义了对maven_ch01的依赖，那么在执行命令时maven会从仓库去找依赖的文件，但是很显然我们的本地仓库中是没有maven_ch01-0.0.1-SWAPSHOT.pom和maven_ch01-0.0.1-SWAPSHOT.jar的，所以最后构建就失败了。

那应该怎么做才能让maven_ch02编译成功呢？

切换到maven_ch01，然后执行maven install命令，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23171244-b9f98131b1fa46d28de62d6b65c29c66.png)
到我们的本地仓库，你会发现maven已经帮我们生成了maven_ch01-0.0.1-SWAPSHOT.jar和maven_ch01-0.0.01-SWAPSHOP.jar，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23171339-f6e39a8bd37a4fafab9e818d514d813e.png)
Ok，再次切换到maven_ch02，并执行mvn compile命令，如图：
![img](http://images.cnitblog.com/blog/548748/201311/23171756-83640e8b0887416bb931032463da5453.png)
看到没有，这次就编译成功了。

maven的install命令真是一个牛逼的命令。
