---
layout: post
title:  mysql group by遇到的问题
description: mysql group by遇到的问题
date:   2014-12-29 16:49:39
categories: blog
tags: mysql group-by 数据库
---
{% highlight sql %}
select userid from t_wxuser group by wxno having count(*) > 1
{% endhighlight %}

像上面这句SQL在oracle中执行就会报错，但mysql不会。  
但是呢，要注意：它只会返回一条数据。这里说的一条不是最终结果只有一条。  
比如说有3个相同wxno的数据（比如wxno=test1），userid分别为101,102,103；还有2个wxno相同的数据（比如test2），userid分别为104,105.  
执行之后返回的数据如下：  
101,104.  

我是在清除t_wxuser表重复数据时发现问题的。执行之后查询发现还有重复的数据，当时以为是sql有问题。  
看了上面这行sql返回的结果才发现是这么回事。  
还有一点呢，你再update/delete一个表的时候，你的from table也是这个表是不行的。  
比如：  
{% highlight sql %}
delete from t_wxuser where userid in (select userid from t_wxuser group by wxno having count(*) >1);
{% endhighlight %}
这样是不行的，会报错。  
错误时这样的：  
[Err] 1093 - You can't specify target table 't_wxuser' for update in FROM clause  

所以呢，没办法加一个别名吧。  
{% highlight sql %}
delete from t_wxuser where USERID in (select * from (select userid from t_wxuser group by wxno having count(*) > 1) aa);
{% endhighlight %}
这种情况在oracle中是不会有问题的。  

还有一点要注意：  
>执行上面的删除语句，并不能删除全部的重复数据。  
**比如：相同userid的数据有3条，执行上面的语句后，还有2条。。**

