---
layout: post
title:  Oracle Copy From用法
description: Oracle Copy From用法
date:   2013-11-18 16:49:39
categories: blog
tags: oracle 数据库 copy from
---
copy from的语法：转[http://blog.163.com/stephen_bk](http://blog.163.com/stephen_bk)
1.语法及使用说明  
1.1 语法  
下面我们来看一下SQL*Copy Command的语法及使用说明。  
在解释SQL*Plus Copy Command的语法之前，我们必须要明确SQL*Plus Copy Command不是一个方法或是函数，也不是一个SQL语句，它是一个命令(command)，当然这个命令必须在SQL*Plus里运行。

SQL*Plus Copy Command的语法：  
COPY {FROM database | TO database | FROM database TO database} {APPEND|CREATE|INSERT|REPLACE} destination_table [(column, column, column, ...)]  
USING query  

我们分部分来解释一下：  
COPY – 这个不太需要解释，主命令，声明要执行COPY操作  
From Database – 源数据库  
To Database – 目标数据库  
此处注意花括号中有三种可选的写法(以”|”隔开)，如果源数据表和目标数据表在同一个Schema中，则可以只写Fro Database，也可以只写To Database，当然还可以是第三种写法，把From Database和To Database写全。
但如果源数据表和目标数据表不在同一个Schema中，则必须用第三种写法，即把From Database和To Database都写全  
From Database和To Database的格式是一样的：USERID/PASSWORD@SID，这个大家都应该很熟悉了。

{APPEND|CREATE|INSERT|REPLACE} – 声明操作数据的方式，下面分别解释一下：  
Append – 向已有的目标表中追加记录，如果目标表不存在，自动创建，这种情况下和Create等效。  
Create – 创建目标表并且向其中追加记录，如果目标表已经存在，则会返回错误。  
Insert – 向已有的目标表中插入记录，与Append不同的是，如果目标表不存在，不自动创建而是返回错误。  
Replace – 用查询出来的数据覆盖已有的目标表中的数据，如果目标表不存在，自动创建。

destination_table – 目标表的名字  
[(column, column, column, ...)] – 可以指定目标表中列的名字，如果不指定，则自动使用Query中的列名。  
USING query – 查询语句，交流的数据来自这儿。

备注:  
1.这种方法适合小额数据的迁移,只能在SQL*PLUS中运行.  
2.这种方法不能copy大对象的表,不然会抛异常"CPY0012: Object datatypes cannot be copied"

**在Copy时分段提交**
设置arraysize和copycommit实现分批提交数据  
参数说明：  
其中arraysize是一批的行数，copycommit是多少批才提交一次。

如设置在copy时10000条数据提交一次  
{% highlight sql %}
SQL> set arraysize 100;  
SQL> set copycommit 100;  
SQL> show arraysize;
arraysize 100
SQL> show copycommit;
copycommit 100
{% endhighlight %}

在SQL*PLUS中动态输入用户名、密码和数据库名？  
执行脚本的方式：@脚本文件全名 参数1的值 参数2的值 参数3的值 参数4的值   
脚本内容如：
{% highlight sql %}
copy from &1/&2@&3 insert error_log using select * from xxx where ...
{% endhighlight %}

注意：copy from脚本必须写在一行。