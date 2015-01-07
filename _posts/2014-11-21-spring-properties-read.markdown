---
layout: post
title:  使用Spring的ReloadableResourceBundleMessageSource读取properties配置
description: 使用Spring的ReloadableResourceBundleMessageSource读取properties配置~~~
date:   2014-11-21 16:49:39
categories: blog
tags: Spring Properties ReloadableResourceBundleMessageSource
---
应用：  
* 1.后台验证提示信息；  
* 2.异常信息。  
spring配置文件如下：
{% highlight xml %}
<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">  
	 <property name="basename" value="classpath:message-resource"/>  
	 <property name="defaultEncoding" value="GBK"/>  
</bean>
{% endhighlight %}

message-resource即为classpath下的message-resource.properties文件。  
接下来定义我们自己的MessageUtil类来使用Spring的MessageSource读取配置。
{% highlight java %}
public class MessageUtil
{
    private static MessageSource messageSource;

    private static void init()
    {
        if (messageSource == null)
        {
            synchronized (MessageUtil.class)
            {
                messageSource = (MessageSource) applicationContextFactory.getBean("messageSource");
            }
        }
    }

    public static String getMessage(String id, Object[] param)
    {
        init();
        return messageSource.getMessage(id, param, "Required", null);
    }
    public static String getMessage(String id)
    {
        init();
        return messageSource.getMessage(id, null, "Required", null);
    }
}
{% endhighlight %}
使用的时候就很简单了。MessageUtil.getMessage(properties文件中配置的key)就OK了。