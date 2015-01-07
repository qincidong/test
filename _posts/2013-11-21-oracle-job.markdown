---
layout: post
title:  Oracle job procedure 存储过程定时任务
description: Oracle job procedure 存储过程定时任务
date:   2013-11-21 16:49:39
categories: blog
tags: oracle job 数据库
---
oracle job有定时执行的功能，可以在指定的时间点或每天的某个时间点自行执行任务。

###一、查询系统中的job，可以查询视图
--相关视图
{% highlight sql %}
select * from dba_jobs;
select * from all_jobs;
select * from user_jobs;
{% endhighlight %}

-- 查询字段描述
/*
<table>
<tr><td>字段（列）         </td><td> 类型           </td><td>      描述                                      </td></tr>
<tr><td>JOB                </td><td>NUMBER          </td><td>任务的唯一标示号                                </td></tr>
<tr><td>LOG_USER           </td><td>VARCHAR2(30)    </td><td>提交任务的用户                                  </td></tr>
<tr><td>PRIV_USER          </td><td>VARCHAR2(30)    </td><td>赋予任务权限的用户                              </td></tr>
<tr><td>SCHEMA_USER        </td><td>VARCHAR2(30)    </td><td>对任务作语法分析的用户模式                      </td></tr>
<tr><td>LAST_DATE          </td><td>DATE            </td><td>最后一次成功运行任务的时间                      </td></tr>
<tr><td>LAST_SEC           </td><td>VARCHAR2(8)     </td><td>如HH24:MM:SS格式的last_date日期的小时，分钟和秒 </td></tr>
<tr><td>THIS_DATE          </td><td>DATE            </td><td>正在运行任务的开始时间，如果没有运行任务则为null</td></tr>
<tr><td>THIS_SEC           </td><td>VARCHAR2(8)     </td><td>如HH24:MM:SS格式的this_date日期的小时，分钟和秒 </td></tr>
<tr><td>NEXT_DATE          </td><td>DATE            </td><td>下一次定时运行任务的时间                        </td></tr>
<tr><td>NEXT_SEC           </td><td>VARCHAR2(8)     </td><td>如HH24:MM:SS格式的next_date日期的小时，分钟和秒 </td></tr>
<tr><td>TOTAL_TIME         </td><td>NUMBER          </td><td>该任务运行所需要的总时间，单位为秒              </td></tr>
<tr><td>BROKEN             </td><td>VARCHAR2(1)     </td><td>标志参数，Y标示任务中断，以后不会运行           </td></tr>
<tr><td>INTERVAL           </td><td>VARCHAR2(200)   </td><td>用于计算下一运行时间的表达式                    </td></tr>
<tr><td>FAILURES           </td><td>NUMBER           </td><td>任务运行连续没有成功的次数                     </td></tr>
<tr><td>WHAT               </td><td>VARCHAR2(2000)   </td><td>执行任务的PL/SQL块                             </td></tr>
<tr><td>CURRENT_SESSION_LABEL </td><td>RAW           </td><td>MLSLABEL 该任务的信任Oracle会话符              </td></tr>
<tr><td>CLEARANCE_HI          </td><td>RAW MLSLABEL  </td><td>该任务可信任的Oracle最大间隙                   </td></tr>
<tr><td>CLEARANCE_LO          </td><td>RAW           </td><td>MLSLABEL 该任务可信任的Oracle最小间隙          </td></tr>
<tr><td>NLS_ENV               </td><td>VARCHAR2(2000)</td><td>任务运行的NLS会话设置                          </td></tr>
<tr><td>MISC_ENV              </td><td>RAW(32)       </td><td>任务运行的其他一些会话参数                     </td></tr>
*/
</table>
{% highlight sql %}
-- 正在运行job
select * from dba_jobs_running;
{% endhighlight %}

其中最重要的字段就是job 这个值就是我们操作job的id号，what 操作存储过程的名称，next_date 执行的时间，interval执行间隔

###二、执行间隔interval 运行频率
<table>
<tr><td>描述                       </td><td>INTERVAL参数值                                                                                    </td></tr>
<tr><td>每天午夜12点               </td><td>TRUNC(SYSDATE + 1)                                                                                </td></tr>
<tr><td>每天早上8点30分            </td><td>TRUNC(SYSDATE + 1) + （8*60+30）/(24*60)                                                          </td></tr>
<tr><td>每星期二中午12点           </td><td>NEXT_DAY(TRUNC(SYSDATE ), ''TUESDAY'' ) + 12/24                                                   </td></tr>
<tr><td>每个月第一天的午夜12点     </td><td>TRUNC(LAST_DAY(SYSDATE ) + 1)                                                                     </td></tr>
<tr><td>每个季度最后一天的晚上11点 </td><td>TRUNC(ADD_MONTHS(SYSDATE + 2/24, 3 ), 'Q' ) -1/24                                                 </td></tr>
<tr><td>每星期六和日早上6点10分    </td><td>TRUNC(LEAST(NEXT_DAY(SYSDATE, ''SATURDAY"), NEXT_DAY(SYSDATE, "SUNDAY"))) + （6×60+10）/（24×60）</td></tr>
</table>

每秒钟执行次
{% highlight sql %}
Interval => sysdate + 1/(24 * 60 * 60)
{% endhighlight %}

如果改成sysdate + 10/(24 * 60 * 60)就是10秒钟执行次

每分钟执行 
{% highlight sql %}
Interval => TRUNC(sysdate,'mi') + 1/ (24*60)
{% endhighlight %}

如果改成TRUNC(sysdate,'mi') + 10/ (24*60) 就是每10分钟执行次

每天定时执行 
例如：每天的凌晨1点执行 
{% highlight sql %}
Interval => TRUNC(sysdate) + 1 +1/ (24)
{% endhighlight %}

每周定时执行 
例如：每周一凌晨1点执行 
{% highlight sql %}
Interval => TRUNC(next_day(sysdate,'星期一'))+1/24
{% endhighlight %}

每月定时执行 
例如：每月1日凌晨1点执行 
{% highlight sql %}
Interval =>TRUNC(LAST_DAY(SYSDATE))+1+1/24
{% endhighlight %}

每季度定时执行 
例如每季度的第一天凌晨1点执行 
{% highlight sql %}
Interval => TRUNC(ADD_MONTHS(SYSDATE,3),'Q') + 1/24
{% endhighlight %}

每半年定时执行 
例如：每年7月1日和1月1日凌晨1点 
{% highlight sql %}
Interval => ADD_MONTHS(trunc(sysdate,'yyyy'),6)+1/24
{% endhighlight %}

每年定时执行 
例如：每年1月1日凌晨1点执行 
{% highlight sql %}
Interval =>ADD_MONTHS(trunc(sysdate,'yyyy'),12)+1/24
{% endhighlight %}

###三、创建job方法
创建job,   
基本语法：
{% highlight sql %}
declare
    variable job number;
begin
    sys.dbms_job.submit(job => :job,
    what => 'prc_name;',
    next_date => to_date('22-11-2013 09:09:41', 'dd-mm-yyyy hh24:mi:ss'),
    interval => 'sysdate+1/86400');--每天86400秒钟，即一秒钟运行prc_name过程一次
    commit;
end;
{% endhighlight %}

使用dbms_job.submit方法过程，这个过程有五个参数：job、what、next_date、interval与no_parse。
{% highlight sql %}
dbms_job.submit( 
job       OUT binary_ineger, 
What      IN  varchar2, 
next_date IN  date, 
interval  IN  varchar2, 
no_parse  IN  booean:=FALSE)
{% endhighlight %}

job参数是输出参数，由submit()过程返回的binary_ineger，这个值用来唯一标识一个工作。一般定义一个变量接收，可以去user_jobs视图查询job值。   
what参数是将被执行的PL/SQL代码块，存储过程名称等。   
next_date参数指识何时将运行这个工作。   
interval参数何时这个工作将被重执行。   
no_parse参数指示此工作在提交时或执行时是否应进行语法分析——true，默认值false。指示此PL/SQL代码在它第一次执行时应进行语法分析，而FALSE指示本PL/SQL代码应立即进行语法分析。

###四、其他job相关的存储过程
在dbms_job这个package中还有其他的过程：broken、change、interval、isubmit、next_date、remove、run、submit、user_export、what；

大致介绍下这些过程：  
1、broken()过程更新一个已提交的工作的状态，典型地是用来把一个已破工作标记为未破工作。这个过程有三个参数：job 、broken与next_date。 
{% highlight sql %}
procedure broken ( 
  job IN binary_integer, 
  broken IN boolean, 
  next_date IN date := SYSDATE 
) 
{% endhighlight %}

job参数是工作号，它在问题中唯一标识工作。 
broken参数指示此工作是否将标记为破——true说明此工作将标记为破，而false说明此工作将标记为未破。 
next_date参数指示在什么时候此工作将再次运行。此参数缺省值为当前日期和时间。

job如果由于某种原因未能成功执行，oracle将重试16次后，还未能成功执行，将被标记为broken，重新启动状态为broken的job，有如下两种方式;   
a、利用dbms_job.run()立即执行该job 
{% highlight sql %}
begin 
  dbms_job.run(:job) --该job为submit过程提交时返回的job number或是去dba_jobs去查找对应job的编号 
end;
{% endhighlight %}

b、利用dbms_job.broken()重新将broken标记为false 
{% highlight sql %}
begin 
  dbms_job.broken (:job, false, next_date) 
end;
{% endhighlight %}

2、change()过程用来改变指定job的设置。 
这个过程有四个参数：job、what 、next_date、interval。 
{% highlight sql %}
procedure change ( 
    job IN binary_integer, 
    what IN varchar2, 
    next_date IN date, 
    interval IN varchar2 
) 
{% endhighlight %}
此job参数是一个整数值，它唯一标识此工作。   
what参数是由此job运行的一块PL/SQL代码块。   
next_date参数指示何时此job将被执行。   
interval参数指示一个job重执行的频度。 

3、interval()过程用来显式地设置重复执行一个job之间的时间间隔数。 
这个过程有两个参数：job、interval。 
{% highlight sql %}
procedure interval( 
    job IN binary_integer, 
    interval IN varchar2 
) 
{% endhighlight %}

job参数标识一个特定的工作。 
interval参数指示一个工作重执行的频度。

4、isubmit()过程用来用特定的job号提交一个job。 
这个过程有五个参数：job、what、next_date、interval、no_parse。 
{% highlight sql %}
procedure isubmit ( 
    job IN binary_ineger, 
    what IN varchar2, 
    next_date IN date, 
    interval IN varchar2, 
    no_parse IN booean := FALSE 
) 
{% endhighlight %}
这个过程与submit()过程的唯一区别在于此job参数作为IN型参数传递且包括一个由开发者提供的job号。如果提供的job号已被使用，将产生一个错误。

5、next_date()过程用来显式地设定一个job的执行时间。这个过程接收两个参数：job、next_date。 
{% highlight sql %}
procedure next_date( 
    job IN binary_ineger, 
    next_date IN date 
) 
{% endhighlight %}

job标识一个已存在的工作。 
next_date参数指示了此job应被执行的日期、时间。

6、remove()过程来删除一个已计划运行的job。这个过程接收一个参数： 
{% highlight sql %}
procedure remove(job IN binary_ineger); 
{% endhighlight %}

job参数唯一地标识一个工作这个参数的值是由为此工作调用submit()过程返回的job参数的值，已正在运行的job不能删除。

7、run()过程用来立即执行一个指定的job。这个过程只接收一个参数： 
{% highlight sql %}
procedure run(job IN binary_ineger) 
{% endhighlight %}

job参数标识将被立即执行的工作。

8、使用submit()过程，job被正常地计划。上面以讲述

9、user_export()过程返回一个命令，此命令用来安排一个存在的job以便此job能重新提交。此程序有两个参数：job、my_call。 
{% highlight sql %}
procedure user_export( 
    job IN binary_ineger, 
    my_call IN OUT varchar2 
) 
{% endhighlight %}
job参数标识一个安排了的工作。 
my_call参数包含在它的当前状态重新提交此job所需要的正文。

10、what()过程应许在job执行时重新设置此正在运行的命令。这个过程接收两个参数：job、what。 
{% highlight sql %}
procedure what ( 
    job IN binary_ineger, 
    what IN OUT varchar2 
) 
{% endhighlight %}
job参数标识一个存在的工作。   
what参数指示将被执行的新的PL/SQL代码。实现的功能：每隔一分钟自动向getSysDate表中插入当前的系统时间。

###五、示例
{% highlight sql %}
/* 每10秒钟执行一次 插入一条时间 */
-- 创建table
create table tab_time(
       current_time timestamp       
);
 
-- 创建存储过程
create or replace procedure pro_job_print
as
   begin
       --dbms_output.put_line('系统时间：' || to_char(sysdate, 'dd-mm-yyyy hh24:mi:ss'));
       insert into tab_time values(sysdate);
   end;
   
-- 调用过程测试   
begin
   pro_job_print;   
end;
 
 
--select 24 * 60 * 60 from dual;   
 
-- 创建job
declare      
   job1 number;
begin
   dbms_job.submit(job1, 'pro_job_print;', sysdate, 'sysdate+10/86400');--每10插入一条记录
end;
 
--相关视图
select * from dba_jobs;
select * from all_jobs;
select * from user_jobs;
-- 正在运行job
select * from dba_jobs_running;
 
 
-- 运行job
begin
   dbms_job.run(26);--和select * from user_jobs; 中的job值对应，看what对应的过程
end; 
 
-- 查询是否插入数据
select to_char(current_time, 'dd-mm-yyyy hh24:mi:ss') current_time from tab_time order by current_time;
 
-- 删除一个job
begin
   dbms_job.remove(26);--和select * from user_jobs; 中的job值对应，看what对应的过程
end;      
{% endhighlight %}
             
###六、关于设置job任务数量和控制并发
初始化相关参数job_queue_processes 
{% highlight sql %}
alter system set job_queue_processes = 39 scope = spfile;//最大值不能超过1000; 
job_queue_interval = 10; //调度作业刷新频率秒为单位

job_queue_process 表示oracle能够并发的job的数量，sqlplus中可以通过语句 
show parameter job_queue_process; 来查看oracle中job_queue_process的值。

select * from v$parameter;

select name, description from v$bgprocess;
{% endhighlight %}

当job_queue_process值为0时表示全部停止oracle的job，可以通过语句 
{% highlight sql %}
alter system set job_queue_processes = 10; 
{% endhighlight %}

来调整启动oracle的job。

如果将job_queue_processes 的值设置为1的话，那就是串行运行，即快速切换执行一个job任务。

###七、job不运行的大概原因
(1)、上面讲解了job的参数：与job相关的参数一个是job_queue_processes，这个是运行job时候所起的进程数，当然系统里面job大于这个数值后，就会有排队等候的，最小值是0，表示不运行job，最大值是1000，在OS上对应的进程时SNPn，9i以后OS上管理job的进程叫CJQn。可以使用下面这个SQL确定目前有几个SNP/CJQ在运行。 
select * from v$bgprocess，这个paddr不为空的snp/cjq进程就是目前空闲的进程，有的表示正在工作的进程。   
另外一个是job_queue_interval，范围在1--3600之间，单位是秒，这个是唤醒JOB的process，因为每次snp运行完他就休息了，需要定期唤醒他，这个值不能太小，太小会影响数据库的性能。

先确定上面这两个参数设置是否正确，特别是第一个参数，设置为0了，所有job就不会自动运行了。

(2)、使用下面的SQL查看job的的broken，last_date和next_date，last_date是指最近一次job运行成功的结束时间，next_date是根据设置的频率计算的下次执行时间，根据这个信息就可以判断job上次是否正常，还可以判断下次的时间对不对，SQL如下： 
select * from dba_jobs;   
有时候我们发现他的next_date是4000年1月1日，说明job要不就是在running，要不就是状态是break(broken=Y)，如果发现job的broken值为Y，找用户了解一下，确定该job是否可以broken，如果不能broken，那就把broken值修改成N，修改再使用上面的SQL查看就发现它的last_date已经变了，job即可正常运行，修改broken状态的SQL如下：
{% highlight sql %}
begin 
    DBMS_JOB.BROKEN(<JOB_ID>, FALSE); 
end;
{% endhighlight %}

(3)、使用下面的SQL查询是否job还在running 
{% highlight sql %}
select * from dba_jobs_running; 
{% endhighlight %}

如果发现job已经Run了很久了还没有结束，就要查原因了。一般的job running时会锁定相关的相关的资源，可以查看一下vaccess和vlocked_object这两个view。如果发现其他进程锁定了与job相关的object，包括package/function/procedure/table等资源，那么就要把其他进程删除，有必要的话，把job的进程也删除，再重新执行看看结果。

(4)、如果上面都正常，但是job还不run，怎么办？那我们要考虑把job进程重启一次，防止是SNP进程死了造成job不跑，指令如下： 
{% highlight sql %}
alter system set job_queue_processes = 0; --关闭job进程，等待5--10秒钟 
alter system set job_quene_processes = 5; --恢复原来的值
{% endhighlight %}

(5)、Oracle的BUG：Oracle9i里面有一个BUG，当计数器到497天时，刚好达到它的最大值，再计数就会变成-1，继续计数就变成0了，然后计数器将不再跑了。如果碰到这种情况就得重启数据库，但是其他的Oracle7345和Oracle8i的数据库没有发现这个问题。

(6)、数据库上的检查基本上就这多，如果job运行还有问题，那需要看一下是否是程序本身的问题，比如处理的资料量大，或者网络速度慢等造成运行时过长，那就需要具体情况具体分析了。我们可以通过下面的SQL手工执行一下job看看： 
{% highlight sql %}
begin 
      dbms_job.run(<job>_ID) 
end; 
{% endhighlight %}

如果发现job执行不正常，就要结合程序具体分析一下。

本文出自： [oracle_procedure_job_interval](http://www.cnblogs.com/hoojo/p/oracle_procedure_job_interval.html) 
