---
layout: post
title:  Eclipse Maven打包出错处理方法
description: Eclipse Maven打包出错处理方法
date:   2013-11-24 16:49:39
categories: blog
tags: maven
---
在执行mvn compile命令时出现下面的错误：  
Please ensure you are using JDK 1.4 or above and not a JRE (the com.sun.tools.javac.Main class is required). 

在CSDN中看到了下面的解决办法：
>这是因为eclipse默认是使用jre作为运行环境，而maven编译需要jdk作为运行环境 
>
>尝试修改eclipse.ini，加入如下语句 
>
>-vm 
>C:\Progra~1\Java\jdk1.6.0_21\bin\javaw.exe 
>
>无效 
>
>仔细看其爆出的提示，似乎将JAVA_HOME环境变量指向jdk目录即可，但依然不起作用。 
>
>其实有个简单办法，就是在eclipse里设置一个jdk的运行环境，然后将当前项目的运行环境设为jdk运行环境即可 
>
>步骤 
>
>window-preferences-java-installed jres 
>
>这里默认有个jre6的JRE定义（maybe你是jre5），一个方法是修改这个jre6，将其location指向你的jdk6目录 
>
>另一个办法是点击Add按钮，选择Standard VM，jre home选择你的jdk6目录。点击finish，这时发现多了一个JRE，将其勾上，以后新的项目，就默认使用这个JRE了 
>
>然后，进入项目的properties页面，选择Java build path，打开libraries标签，remove默认的jre6，add Libraries，选择JRE system library，选择你刚创建的jdk（已被默认选中），finish 
>
>现在运行maven 的编译，一切正常。 
>============================================================================ 
>m2eclipe 经常会报这个错，原因是对于安装了JDK的机器，会有两个jre，一个在C:/Program Files/Java/jre6下，一个在C:Program FilesJavajdk1.6.0_20jre, 而默认eclipse如果不做改变，会使用前者，而m2eclipse默认会去找JDK下的jre 
>解决办法： 
>在eclipse.ini中添加两行 
>    -vm 
>    C:/Program Files/Java/jdk1.6.0_16/bin/javaw.exe 
>注意: 要写在两行，写在一行不能生效 
>注意: 这两行要定在-vmargs之前，不然也不能生效 
>注意: 最后一行也可以写成C:/Program Files/Java/jdk1.6.0_16/bin/ 
>好了，不出意外，重新启动eclipse,应该会好。但是如果有意外，你会启动不起来eclipse，并且会报错“could not create the java virtual machine ”.

按照上面的2点设置之后就Ok了。
