---
layout: post
title:  记录一个使用Spring注解事务管理遇到的问题
description: 使用Spring注解管理事务必须注意始终只能有一个事务。因为有时候很难发现问题。
date:   2014-12-29 16:49:39
categories: blog
tags: Spring 事务 注解
---
在Service方法上加了如下注解：
{% highlight java %}
Transactional(rollbackFor=Exception.class,timeout=30)
{% endhighlight %}

而且spring的事务管理配置也配置了。

在Service方法中有一个批量保存操作，一个更新操作。
在保存操作后，手动抛出了一个异常，发现事务没有回滚。

而且，我确定我用另外一个测试Service方法测试过，是OK的，事务会回滚。
那到底是何原因呢？

**跟踪发现，是批量保存的问题。第一个保存是批量保存，跟踪代码发现在批量保存中重新开启了一个事物。**

批量保存的代码如下：
{% highlight java%}
//批量保存
public void batchSave(List<E> list){
	StatelessSession session = getSessionFactory().openStatelessSession();
	Transaction tx = session.beginTransaction();
	tx.begin();
	
	for(E e : list){
		session.insert(e);
	}
	
	tx.commit();
	session.close();
	
	logger.info("batch save success");
}
{% endhighlight %}