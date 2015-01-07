---
layout: post
title:  Java注解
description: Java注解是在JDK1.5引入的，以@注解名存在代码中。<br>声明一个注解使用@interface，其中每个方法实际上是声明了一个配置参数，方法名即参数名，返回类型即参数类型。<br>通过default来设置默认值。<br>@Retention和@Target用来声明注解的本身的行为。@Retention指明注解保留策略，有RetentionPolicy.SOURCE（源码）、RetentionPolicy.RUNTIME（JVM运行时）、RetentionPolicy.CLASS（类文件）3种。只有当声明为RUNTIME时才可以通过java的反射API来获取注解信息。<br>@Target指明了注解作用的类型，有包、类、方法、构造方法、字段、本地变量、方法参数等。所有的注解都是自动继承Annotation的。
date:   2014-11-25 16:49:39
categories: blog
tags: java 注解
---
Java注解是在JDK1.5引入的，以@注解名存在代码中。  
声明一个注解使用@interface，其中每个方法实际上是声明了一个配置参数，方法名即参数名，返回类型即参数类型。  
通过default来设置默认值。  
@Retention和@Target用来声明注解的本身的行为。@Retention指明注解保留策略，有RetentionPolicy.SOURCE（源码）、RetentionPolicy.RUNTIME（JVM运行时）、RetentionPolicy.CLASS（类文件）3种。只有当声明为RUNTIME时才可以通过java的反射API来获取注解信息。  
@Target指明了注解作用的类型，有包、类、方法、构造方法、字段、本地变量、方法参数等。所有的注解都是自动继承Annotation的。  

###包类型的注解
定义一个包类型的注解：
{% highlight java %}
package com.ant.test.annotation;
 
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
 
/**
* @Description: TODO
* @author luckystar2008
* @date 2013-8-19 下午2:33:50
*/
@Target(ElementType.PACKAGE)
@Retention(RetentionPolicy.RUNTIME)
public @interface PkgAnnotation {
   String description();
}
{% endhighlight %}

然后再某个类的Package声明上加上这个注解。加了之后，编译器立马显示错误。具体提示信息如下：  
>Package annotations must be in file package-info.java.  
这个错误信息相当明显，即包类型的注解需要在package-info.java文件中。  
Ok，好吧，创建一个package-info.java文件，用Eclipse创建发现是没法创建的，提示Type name is not valid. The type name 'package-info' is not a valid identifier。  
得，咱用记事本创建，Ok，创建了一个package-info.java:
{% highlight java %}
@PkgAnnotation
package com.ant.test.annotation;
{% endhighlight %}

Ok ,我们在com.ant.tets.annotation包上应用注解。这次编译没有报错。  
在上面，这个包类型的注解我们指定了是RetentionPolicy.RUNTIME，因此，可以通过Java的发射API获取包com.ant.test.annotation上的所有注解。
{% highlight java %}
package com.ant.test.annotation;
 
import java.lang.annotation.Annotation;
import org.apache.log4j.Logger;
 
/**
* @Description: TODO
* @author luckystar2008
* @date 2013-8-19 下午1:55:14
* @version V1.0
*/
public class Bean {
    private Logger logger = Logger.getLogger(Bean.class);
 
    public void testPkgAnnotation1() {
       logger.info("测试包类型的注解。。。");
       String pkgName = "com.ant.test.annotation";
       Package pkg = Package.getPackage(pkgName);
       Annotation[] annotations = pkg.getAnnotations();
       for (Annotation annotation:annotations) {
           if (annotation instanceofPkgAnnotation) {
              logger.info("This is PkgAnnotation.");
              logger.info("package description:" +    ((PkgAnnotation)annotation).description());
           }
       }
   }
 
    public static void main(String[] args) {
       Bean b = new Bean();
       b.testPkgAnnotation1();
    }
}
{% endhighlight %}

输出：  
>[08/19 16:59:50] [INFO] Bean: 测试包类型的注解。。。  
>[08/19 16:59:50] [INFO] Bean: This is PkgAnnotation.  
>[08/19 16:59:50] [INFO] Bean: package description:This is the test msg for this pacakge.  

TIP:package-info.java是一个比较特殊的文件，平时没人会关注到它。  
在Spring的源文件中，我们可以看到package-info.java的影子，它就是用来描述一个包的。其次，就是上面所看到的，为标注在包上的注解提供便利。  
在package-info.java中还可以创建具有包级访问权限的类和常量，当然，类名也不可能是public的，否则你的文件名就得是类名了。  
如下，在package-info中加了2个类，一个Test，一个PkgConstants。
{% highlight java %}
@PkgAnnotation(description = "This is the test msg for this pacakge.")
package com.ant.test.annotation;
 
class Test {
   public void test() {
       System.out.println("This is a test method in package.");
   }
}
 
class PkgConstants {
    public static String PACKAGE_NAME = com.ant.test.annotation.PkgConstants.class.getPackage().getName();
}
{% endhighlight %}

这2个类都是在同一个包下的类可以访问的。  
在Bean.java中另加一个方法来测试：
{% highlight java %}
public void testPkgAnnotation2() {
   logger.debug("测试package-info.java中的类和常量。");

   Test  test = new Test();
   test.test();

   String packageName = PkgConstants.PACKAGE_NAME;
   logger.info("packagename:" + packageName);
}
{% endhighlight %} 

运行，程序输出：  
>08/19 17:14:53] [DEBUG] Bean: 测试package-info.java中的类和常量。  
>This is a test method in package.  
>[08/19 17:14:53] [INFO] Bean: packagename:com.ant.test.annotation  

###1.方法上的注解  
下面定义一个作用于方法上的注解：
{% highlight java %}
package com.ant.test.annotation;
 
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
 
/**
* @Description: TODO
* @author luckystar2008
* @date 2013-8-19 下午1:54:08
*/
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnno {
   int value() default 0;
   String name();
   Color color() default Color.BLUE;
   String[] className() default {};
   enum Color{RED,BLUE,GREEN};
}
{% endhighlight %}

如上，int value() default 0这里，如果在注解中指定了value()，如果没有其他的方法，那么在使用注解的时候，你可以直接用注解的名字就行了，如@MyAnno。  
注解中指定了默认值的参数则在使用时不指定值的情况下会使用默认值。  
下面是测试方法：
{% highlight java %}
@MyAnno(name = "abc")
public void testMyAnno1() {
    logger.info("测试自定义注解。。。");
       Method method = null;
　　try {
           method = Bean.class.getMethod("testMyAnno1", null);
       } catch (SecurityException e) {
　　　　　　// TODO Auto-generated catch block
           e.printStackTrace();
       } catch (NoSuchMethodException e) {
　　　　　　// TODO Auto-generated catch block
           e.printStackTrace();
       }
 
    MyAnno ma = method.getAnnotation(MyAnno.class);
    logger.info("value:" + ma.value());
    logger.info("name:" + ma.name());
    logger.info("color:" + ma.color());
    logger.info("className:" + (ma.className().length == 0 ? "{}":ma.className()));
}
{% endhighlight %}

因为注解中只有name没有指定默认值，因此这里只需要对name指定值。

输出：  
>[08/20 09:23:37] [INFO] Bean: 测试自定义注解。。。  
>[08/20 09:23:38] [INFO] Bean: value:0  
>[08/20 09:23:38] [INFO] Bean: name:abc  
>[08/20 09:23:38] [INFO] Bean: color:BLUE  
>[08/20 09:23:38] [INFO] Bean: className:{}

从输出我们可以看到，除了name外其他的都是使用的默认值。

###2.类上的注解
定义一个作用于类上的注解：
{% highlight java %}
package com.ant.test.annotation;
 
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
 
/**
* @Description: TODO
* @author luckystar2008
* @date 2013-8-20 上午9:26:20
*/
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ClassAnnotation {
    String url();
    RequestMethod method() default RequestMethod.POST;
    enum RequestMethod {
    GET,POST
    }
}
{% endhighlight %}

测试类：
{% highlight java %}
package com.ant.test.annotation;
 
import org.apache.log4j.Logger;
import com.ant.test.annotation.ClassAnnotation.RequestMethod;
/**
* @Description: TODO
* @author luckystar2008
* @date 2013-8-20 上午9:28:21
*/
@ClassAnnotation(url="/jsp/index.htm",method=RequestMethod.POST)
public class TestClassAnnotation {
    private static Logger logger = Logger.getLogger(TestClassAnnotation.class);
    public static void main(String[] args) {
        logger.info("Test Annotation Effect On Class.");
        ClassAnnotation ca = TestClassAnnotation.class.getAnnotation(ClassAnnotation.class);
        logger.info("url:" + ca.url());
        logger.info("method:" + ca.method().name());
   }
}
{% endhighlight %}

输出：  
>[08/20 09:46:00] [INFO] TestClassAnnotation: Test Annotation Effect On Class.  
>[08/20 09:46:00] [INFO] TestClassAnnotation: url:/jsp/index.htm  
>[08/20 09:46:00] [INFO] TestClassAnnotation: method:POST  

###3.注解的继承
注解是可以继承的，当然必须在注解上指定@Inherited。  
还是那#3中的例子：  
首先，在注解上加上@Inherited，表示可以被继承。
{% highlight java %}
package com.ant.test.annotation;
 
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
 
/**
* @Description: TODO
* @author luckystar2008
* @date 2013-8-20 上午9:26:20
*/
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface ClassAnnotation {
   String url();
   RequestMethod method() default RequestMethod.POST;
   enum RequestMethod {
       GET,POST
   }
}
{% endhighlight %}

然后，定义一个类，在类上使用注解。
{% highlight java %}
package com.ant.test.annotation;
 
import com.ant.test.annotation.ClassAnnotation.RequestMethod;
/**
* @Description: TODO
* @author luckystar2008
* @date 2013-8-20 上午9:52:30
* @version V1.0
*/
@ClassAnnotation(url="/jsp/index.do",method=RequestMethod.POST)
public class ClassAnno {
}
{% endhighlight %}

最后定义一个子类继承ClassAnno。
{% highlight java %}
package com.ant.test.annotation;
 
import java.lang.annotation.Annotation;
/**
* @Description: TODO
* @author luckystar2008
* @date 2013-8-20 上午9:51:44
*/
public class InheritClass extends ClassAnno {
      public static void main(String[] args) {
          Annotation[] anns = InheritClass.class.getAnnotations();
          for (Annotation ann:anns) {
              System.out.println(ann);
          }
      }
}
{% endhighlight %}

输出：  
>@com.ant.test.annotation.ClassAnnotation(method=POST, url=/jsp/index.do)

如果你不在注解上加@Inherited，那么这里也不会有任何输出了。  
当然，注解还可以作用与字段、构造方法、变量、方法参数等上面。  
Spring的MVC中就使用了作用于类上的@Controller，作用于方法上的@RequestMapping，作用于方法参数的@ModelAttribute等。  
通过指定注解的@Retention(RetentionPolicy.RUNTIME)，通过反射API获取到注解的值，这样就可以改变对象的行为。通过在方法上使用权限注解，达到细粒度的控制权限等。  
相对于XML配置和注解配置，我个人更喜欢注解，注解显的很简洁，优雅.
