# MySQL 8. 0 性能优化 25 讲  

## 姚远 


### MySQL 8 性能优化系列课程内容
```
1. 数据库优化的重要性
2. 设置系统参数
3. 内存的分配
4. InnoDB日志的配置
5. innodb_dedicated_server参数
6. 硬盘读写参数
7. 其他参数
8. 资源组
9. 根据最繁忙线程找出TOP SQL
10 .慢查询日志找出TOP SQL
11 .性能视图找出TOP SQL
12 .sys数据库中的存储过程找出TOP SQL
13 .SQL语句的执行计划
14 .SQL执行性能的评估
15 .解密MySQL的优化器
16 .使用hint改变执行计划
17 .InnoDB的主键和二级索引
18 .优化索引
19 .覆盖索引使分页查询性能提高 30 倍
20 .统计信息
21 .直方图拯救低效率的SQL
22 .多表的连接优化
23 .如何让排序速度成倍提高
24 .表空间碎片整理
25 .让SQL优雅且高效的CTE
持续更新中......
```

### 讲师介绍

姚远
Oracle ACE(http://apex.oracle.com/pls/apex/aces/d/yuan.yao)，华为云MVP。Oracle
10 g、 12 c OCM, MySQL 5. 6 、 5. 7 、 8. 0 OCP，并在：EMC、IBM p、RedHat、Cisco、
SQL Server、DB 2 等领域拥有 20 + 技术认证。
部分勋章：https://www.credly.com/users/yuan-yao. 296 cfc 38 /badges 。
曾任IBM公司数据库部门经理。两次获得国家部级科技进步奖，发明过两项计算机专利。微信号：
yaoyuanace，个人公众号：数据慧眼。




### 视频课程资料

博客：https://yaoyuan.blog.csdn.net
视频课程: 斗音：姚远ACE B站：姚远ACE
讲义下载：关注我的个人公众号“数据慧眼”可以得到我的讲义下载链接，讲义内容后续将持续更新。



### MySQL 8 优化 —— 实例优化和参数设置


### 实例优化和参数设置

数据库优化的重要性
设置系统参数
内存的分配
InnoDB日志的配置
innodb_dedicated_server参数
硬盘读写参数
其他参数
资源组


### 实例优化和参数设置

数据库优化的重要性
设置系统参数
内存的分配
InnoDB日志的配置
innodb_dedicated_server参数
硬盘读写参数
其他参数
资源组


### 应用服务器的性能特点

应用服务器通常消耗的是 CPU 和内存，对磁盘 IO 需求很小。
纵向扩展(增加网络、硬盘、内存和 CPU 的处理能力)应用服务器可以线性增加性能。
水平扩展(增加服务器数量)难度小，因为应用服务器之间通常只共享数据库。
水平扩展也会线性增加性能，通常服务器数量增加 N 倍，性能也会增加 N 倍。


### 数据库服务器的性能特点

数据库服务器的瓶颈通常在磁盘的读写。
纵向扩展(增加网络、硬盘、内存和 CPU 的处理能力)数据库服务器可以线性增 加性能。
水平扩展(增加服务器数量)难度很大，例如将数据库服务器数量从一台增加到两台，运维的难度可
能增加了 10 倍，因为涉及多台服务器之间的并发时的锁和数据同步的问题。
水平扩展可能不会带来性能的线性增加，例如数据库服务器数量增加 1 倍，性能可能只会增加 0. 5
到 0. 8 倍。


### 数据库优化的重要性

应用服务器扩展容易，而数据库服务器扩展困难。
因此当系统遇到性能瓶颈时，数据库的性能优化就显得非常重要，并且回报丰厚。
目前在项目的开发设计阶段研发人员主要关注功能的实现，对性能缺乏足够的重视，等投产后，随
着业务量和数据量的增加，性能瓶颈就暴露出来了。
因此这个时候数据库优化的空间都很大，很多系统经过优化后，性能可以提高 10 倍到 100 倍。其
中 SQL 语句的优化特别明显，一些需要运行数天或数小时的 SQL 语句， 经过优化后，可以在几
分钟甚至几秒钟内完成。


### 实例优化和参数设置

数据库优化的重要性
设置系统参数
内存的分配
InnoDB日志的配置
innodb_dedicated_server参数
硬盘读写参数
资源组


### 系统参数的说明

MySQL的系统参数当然是对MySQL性能影响最大的因素。
MySQL的系统参数保存在参数文件中，不同平台的参数文件位置不一样，RedHat Linux中的默认
参数文件位置是 /etc/my.cnf，Debian Linux中的默认参数文件位置是/etc/mysql/my.cnf。
MySQL 8 一共有六百多个系统参数，但对系统性能影响大的参数只有几十个。
参数说明参见：https://dev.mysql.com/doc/refman/ 8. 0 /en/server-system-variable-
reference.html


### 全局级和会话级参数

MySQL的系统参数根据作用范围可以分为全局级和会话级，全局级的系统参数对实例的所有会话起
作用，会话级的系统参数只对当前会话起作用，在这两个级别修改系统参数的例子如下：

mysql> set session binlog_rows_query_log_events=on;
mysql> set global binlog_rows_query_log_events=on;
其中session可以省略。


### 修改全局系统参数的生效时间

修改全局系统参数只对修改后连接到MySQL的会话生效，在修改之前已经建立的会话还保持原来的参数值，例如下面的命令
修改一个全局系统参数：
mysql> set global sort_buffer_size= 16 * 1024 * 1024 ;
下面的命令检查这个系统参数的全局级和会话级的值：
mysql> select @@global.sort_buffer_size,@@session.sort_buffer_size;
+---------------------------+----------------------------+
|@@global.sort_buffer_size|@@session.sort_buffer_size|
+---------------------------+----------------------------+
| 16777216 | 262144 |
+---------------------------+----------------------------+
1 row in set ( 0. 00 sec)
发现全局级的参数值已经改过来了，而会话级的参数值没有变。


### 一些系统参数只能是全局级的

有一些系统参数只能是全局级的，在会话级修改这类参数会出错，例如：

mysql> set log_error_verbosity= 2 ;
ERROR 1229 (HY 000 ): Variable 'log_error_verbosity' is a GLOBAL variable and should be
set with SET GLOBAL
参见参数说明（https://dev.mysql.com/doc/refman/ 8. 0 /en/server-system-variable-
reference.html）中的Var Scope字段。


### 查询其他会话的参数

如果要查询其他会话的参数可以在performance_schema.variables_by_thread视图中查询，例如下面的
SQL语句查询所有会话的事务隔离级别如下：
mysql> select * from performance_schema.variables_by_thread where
variable_name='transaction_isolation';
+-----------+-----------------------+-----------------+
|THREAD_ID|VARIABLE_NAME|VARIABLE_VALUE|
+-----------+-----------------------+-----------------+
| 60 |transaction_isolation|REPEATABLE-READ|
| 67 |transaction_isolation|SERIALIZABLE|
+-----------+-----------------------+-----------------+
2 rowsinset( 0. 00 sec)


### 静态参数和动态参数

MySQL的系统参数还可以分为静态参数和动态参数，动态参数可以MySQL运行中进行修改，静态
参数在MySQL启动后无法修改，例如：

mysql> set auto_generate_certs=on;
ERROR 1238 (HY 000 ): Variable 'auto_generate_certs' is a read only variable


### 系统参数设置为 default

如果把会话级系统参数设置为default，对应的是全局级系统参数值。下面两个设置会话级参数的语
句效果是一样的：

mysql> SET @@SESSION.max_join_size = DEFAULT;
mysql> SET @@SESSION.max_join_size = @@GLOBAL.max_join_size;
如果把全局级系统参数设置default，将把系统参数恢复为MySQL内置的默认值，而不是像很多人
认为的是参数文件里面的设置值。


### 持久化参数设置

在系统参数设置时，一个容易犯的错误是在MySQL运行时修改了参数值，但没有同时修改参数文件
里面的配置，当MySQL重新启动后，参数文件里的旧值生效，之前的修改丢掉了。在MySQL 8 里，
MySQL推出了让参数持久化的命令，可以让在联机时修改的系统参数在重新启动后仍然生效，例如：

mysql> set persist max_connections = 1000 ;
或者：
mysql> set @@persist.max_connections = 1000 ;
如果想让系统参数在本次MySQL运行时不生效，只是在下次启动时生效，可以使用下面的命令：
mysql> set persist_only back_log = 100 ;
或者：
mysql> set @@persist_only.back_log = 100 ;


### 持久化的系统参数以 JSON 格式保存

持久化的系统参数以JSON格式保存在数据目录的mysqld-auto.cnf文件中，例如：
$ cat /var/lib/mysql/mysqld-auto.cnf
{ "Version" : 1 , "mysql_server" : { "max_connections" : { "Value" : " 1000 " , "Metadata" :
{ "Timestamp" : 1624532357790912 , "User" : "root" , "Host" : "localhost" } } ,
"mysql_server_static_options" : { "back_log" : { "Value" : " 100 " , "Metadata" :
{ "Timestamp" : 1624533604896754 , "User" : "root" , "Host" : "localhost" } } } } }
可以通过reset persist命令来清除mysqld-auto.cnf文件中的所有配置，也可以通过reset persist
接参数名的方式来清除某个指定的配置参数。


### 参数值的来源

现查在询系视统图参pe数rfo可rm以a从n多ce个_s来ch源e进ma行.v设ar置ia，bl有es时_i分nf不o找清到参相数关值信到息底，来例自如哪：里，到底那种方式的设置在起作用，这时可以
mysql> select variable_name, variable_source as source,
variable_path, set_time, set_user as user, set_host
from performance_schema.variables_info
where variable_name='max_connections' or variable_name='socket'\G
*************************** 1. row ***************************
variable_name: max_connections
source: DYNAMIC
variable_path:
set_time: 2021 - 08 - 06 14 : 30 : 07. 128393
user: root
set_host: localhost
*************************** 2. row ***************************
variable_name: socket
source: GLOBAL
variable_path: /etc/my.cnf - - 参数文件名和路径
set_time: NULL
user: NULL
set_host: NULL

2 rows in set ( 0. (^01) 姚 s远ec ) 微信号：yaoyuanace 个人公众号：数据慧眼


### 查询参数文件

MySQL localhost SQL > select variable_path,variable_source,count(*) from
performance_schema.variables_info where length(variable_path)!= 0 group by
variable_path,variable_source;
+--------------------------------------------------------+-----------------+----------+
|variable_path|variable_source|count(*)|
+--------------------------------------------------------+-----------------+----------+
|/root/mysql-sandboxes/ 3320 /my.cnf|EXPLICIT| 23 |
|/root/mysql-sandboxes/ 3320 /sandboxdata/mysqld-auto.cnf|PERSISTED| 1 |
+--------------------------------------------------------+-----------------+----------+


### 实例优化和参数设置

数据库优化的重要性
设置系统参数
内存的分配
InnoDB日志的配置
innodb_dedicated_server参数
硬盘读写参数
其他参数
资源组


### 计算 MySQL 在负载高峰时占用的总内存

mysql> select ( @@key_buffer_size + @@innodb_buffer_pool_size +
@@innodb_log_buffer_size + @@binlog_cache_size+ @@max_connections
*( @@read_buffer_size + @@read_rnd_buffer_size + @@sort_buffer_size +
@@join_buffer_size + @@thread_stack + @@tmp_table_size ) ) / ( 1024 * 1024 * 1024 )
as max_memory_gb;
在实际工作中，这里算出来的数值通常偏大，因为所有的线程都同时用到设定内存分配 的最大值的
情况几乎不会出现，每个线程如果只是处理简单的工作，大约只需要 256 KB 的 内存。通过查询
sys.memory_global_total 视图可以得到当前 MySQL 实例使用内存总和。


### 系统参数 key_buffer_size

系统参数 key_buffer_size 从字面上理解是指定索引缓存的大小，需要注意的是它只对 MyISAM
表起作用，对 InnoDB 表无效。这个参数在字面上并没有明确加上 MyISAM，是因 为它是在
MyISAM 作为 MySQL 默认存储引擎的时代产生的。由于现在通常用的是 InnoDB 表，因此通常
不需要调整这个参数。


### 参数 innodb_buffer_pool_size

MySQL 的默认配置是针对内存为 512 MB 的虚拟机设计的，innodb_buffer_pool_size 默认值是
128 MB，这个值在生产中通常都太小。
当一台服务器被一个 MySQL 实例独占 时，通常 innodb_buffer_pool_size 可以设置为内存的
70 %左右。
如果在同一台服务器上还有 其他的 MySQL 或别的应用，设置 innodb_buffer_pool_size 的大小
就要考虑更多因素，一个重要的因素是 InnoDB 的总数据量(包括表和索引)。


### InnoDB 的总数据量 ( 包括表和索引 )

mysql> SELECT count(*) as TABLES, concat(round(sum(table_rows)/ 1000000 , 2 ),'M')
num_row,
concat(round(sum(data_length)/( 1024 * 1024 * 1024 ), 2 ),'G') DATA,
concat(round(sum(index_length)/( 1024 * 1024 * 1024 ), 2 ),'G') idx,
concat(round(sum(data_length+index_length)/( 1024 * 1024 * 1024 ), 2 ),'G') total_size
FROM information_schema.TABLES WHERE engine='InnoDB';
把参数 innodb_buffer_pool_size 设置成超过 InnoDB 的总数据量是没有意义的，通常设置到能
容纳 InnoDB 的活跃数据就够了。


### InnoDB 缓存池的命中率

两个MySQL的状态参数可以计算出它的命中率：
（ 1 ）Innodb_buffer_pool_read_requests：表示向InnoDB缓存池进行逻辑读的次数。
（ 2 ）Innodb_buffer_pool_reads：表示从物理磁盘中读取数据的次数。
InnoDB缓存池的命中率的计算公式如下：
InnoDB缓存池的命中率=（Innodb_buffer_pool_read_requests - Innodb_buffer_pool_reads）
/ Innodb_buffer_pool_read_requests * 100 %。


### InnoDB 缓存池的命中率的计算例子

mysql>showstatuslike'Innodb_buffer_pool_read%s';
+----------------------------------+---------+
|Variable_name|Value|
+----------------------------------+---------+
|Innodb_buffer_pool_read_requests| 1059322 |
|Innodb_buffer_pool_reads| 6091 |
+----------------------------------+---------+
2 rowsinset( 0. 00 sec)

mysql>select( 1059322 - 6091 )/ 1059322 * 100 'InnoDBbufferpoolhit';
+------------------------+
|InnoDBbufferpoolhit|
+------------------------+
| 99. 4250 |
+------------------------+

```
姚远 微信号：yaoyuanace 个人公众号：数据慧眼
```

### 状态参数 Innodb_buffer_pool_reads

代表MySQL不能从InnoDB缓存池读到需要的数据而不得不从硬盘中进行读的次数，使用下面的命令查询
MySQL每秒从磁盘读的次数：
$ mysqladmin extended-status - ri 1 | grep Innodb_buffer_pool_reads
| Innodb_buffer_pool_reads | 1476098323 |
| Innodb_buffer_pool_reads | 734 |
| Innodb_buffer_pool_reads | 987 |
| Innodb_buffer_pool_reads | 595 |
把这个值和硬盘的I/O能力进行对比，如果接近了硬盘处理I/O的上限，那么从操作系统层查看到的CPU用于等
待I/O的时间（IO wait，例如vmstat中的cpu的wa或iostat中的%iowait）会变长，这时硬盘I/O就成了性能的
瓶颈，增大InnoDB缓存池可能会减少MySQL访问硬盘的次数，提高数据库的性能。


### 对设置 InnoDB 缓存大小的考虑

太小的缓冲池可能会导致数据页被频繁地从磁盘读取到内存，引起性能下降。
但如果设置得过大，又可能会造成内存被交换到位于硬盘的内存交换分区，引起性能急剧下降。
这两种情况比较起来，把InnoDB缓存池设置小一些对性能的负面影响并不特别大。实际生产中，
mysqld进程崩溃的一个常见原因是操作系统的内存耗尽，操作系统被迫把mysqld进程杀死。


### 设置 InnoDB 缓存大小

早期调整innodb_buffer_pool_size需要重新启动MySQL，从MySQL 5. 7 后，这个参数可以动态
地进行调整，例如下面的命令把这个参数设置成 256 MB：

mysql> set persist innodb_buffer_pool_size = 256 * 1024 * 1024 ;


在 **MySQL** 的错误日志中可以看到 **MySQL** 调整 **InnoDB** 缓存池的过程

2021 - 06 - 26 T 06 : 58 : 26. 824890 Z 6042 [Note] [MY- 012398 ] [InnoDB] Requested to resize buffer pool. (new size: 268435456 bytes)
2021 - 06 - 26 T 06 : 58 : 26. 866619 Z 0 [Note] [MY- 011880 ] [InnoDB] Resizing buffer pool from 134217728 to 268435456 (unit= 134217728 ).
2021 - 06 - 26 T 06 : 58 : 26. 892090 Z 0 [Note] [MY- 011880 ] [InnoDB] Disabling adaptive hash index.
2021 - 06 - 26 T 06 : 58 : 26. 899180 Z 0 [Note] [MY- 011885 ] [InnoDB] disabled adaptive hash index.
2021 - 06 - 26 T 06 : 58 : 26. 899225 Z 0 [Note] [MY- 011880 ] [InnoDB] Withdrawing blocks to be shrunken.
2021 - 06 - 26 T 06 : 58 : 26. 899251 Z 0 [Note] [MY- 011880 ] [InnoDB] Latching whole of buffer pool.
2021 - 06 - 26 T 06 : 58 : 26. 899294 Z 0 [Note] [MY- 011880 ] [InnoDB] buffer pool 0 : resizing with chunks 1 to 2.
2021 - 06 - 26 T 06 : 58 : 26. 918099 Z 0 [Note] [MY- 011891 ] [InnoDB] buffer pool 0 : 1 chunks ( 8192 blocks) were added.
2021 - 06 - 26 T 06 : 58 : 26. 920019 Z 0 [Note] [MY- 011894 ] [InnoDB] Completed to resize buffer pool from 134217728 to 268435456.
2021 - 06 - 26 T 06 : 58 : 26. 920066 Z 0 [Note] [MY- 011895 ] [InnoDB] Re-enabled adaptive hash index.
2021 - 06 - 26 T 06 : 58 : 26. 920136 Z 0 [Note] [MY- 011880 ] [InnoDB] Completed resizing buffer pool at 210626 4 : 28 : 26.


### innodb_buffer_pool_instances 系统参数

一个和相关innodb_buffer_pool_size的参数是innodb_buffer_pool_instances，它设定把
InnoDB缓存池分成几个区，当innodb_buffer_pool_size大于 1 GB时，这个参数才会起作用，对于
大的InnoDB缓存池，建议把它设置得大一些，这样可以减少获取访问InnoDB缓存池时需要上锁的
粒度，以提高并发度。


### 实例优化和参数设置

数据库优化的重要性
设置系统参数
内存的分配
InnoDB日志的配置
innodb_dedicated_server参数
硬盘读写参数
其他参数
资源组


### InnoDB 日志

InnoDB日志保存着已经提交的数据变化，用于在崩溃恢复时把数据库的变化恢复到数据文件，除了
崩溃恢复，其他时候都不会读日志文件。向日志文件写数据的方式是顺序写，这比离散写的效率要
高很多，而向数据文件写数据通常是离散写比较多。
日志缓冲区是一个内存缓冲区，InnoDB使用它来缓冲重做日志事件，然后再将其写入磁盘。日志缓
冲区的大小由系统参数innodb_log_buffer_size控制，默认是 16 MB，在大多数情况下是够用的。
如果有大型事务或大量较小的并发事务，可以考虑增大innodb_log_buffer_size，这个参数在
MySQL 8 中可以动态设置。
默认在datadir下有两个 48 MB的日志文件ib_logfile 0 和ib_logfile 1 。


### 日志产生量

InnoDB的日志产生量是衡量数据库繁忙程度的重要指标，也是设置日志文件大小的依据。查询日志
产生量的相关信息有两个方法。
第一个方法是查询information_schema.innodb_metrics或sys.metrics视图中的对应计量值。
第二个方法是使用show engine innodb statu命令查询日志产生量的相关信息，这些信息在输出的
LOG部分，这种方法不需要激活InnoDB中的相关计量。


### 查询日志视图

使用下面的命令可以激活这些计量：
mysql> set global innodb_monitor_enable = 'log_lsn_%';
激活后，一个查询结果的例子如下：
mysql>selectname,count,statusfrominformation_schema.innodb_metricswherenamelike'log_lsn%';
+--------------------------------+-----------+---------+
|name|count|STATUS|
+--------------------------------+-----------+---------+
|log_lsn_last_flush| 421908956 |enabled|
|log_lsn_last_checkpoint| 404648325 |enabled|
|log_lsn_current| 421909901 |enabled|
|log_lsn_archived| 0 |enabled|
|log_lsn_checkpoint_age| 17261576 |enabled|
|log_lsn_buf_dirty_pages_added| 421909901 |enabled|
|log_lsn_buf_pool_oldest_approx| 408201960 |enabled|
|log_lsn_buf_pool_oldest_lwm| 406104808 |enabled|
+--------------------------------+-----------+---------+
8 rowsinset( 0. 00 sec)
这为里对的日l志og文_件lsn的_写ch入e是ck循po环in覆t_盖ag的e，是检当查前点日之志前量的减日去志最都近已一经次写检入查数点据的文日件志了量，，不等再于需lo要g_了ls，n_可cu以r被re覆nt盖减。去这log里_看ls到n_的la日st志_c文h件ec的kp使oi用nt量，大也约就是是 1 日 7 志MB文。件的使用量，因


#### 使用 show engine innodb statu 查询日志产生量

m...ysql> show engine innodb status\G

- L-O-G
- L-o-g sequence number 425640652

LLoogg bbuuffffeerr acsosmigpnleetde (^) du pu (^) pt ot (^) o (^4 42255664400665522)
LLoogg wfluristtheend uupp ttoo (^442255664309695724)
APdadgeeds (^) fdliurstyh (^) epda gueps t uop t o 4 40275063460166562
L 2 a 5 s 2 t 8 2 c 3 h elcokgp io/ion'st adto (^) n (^) e (^) , (^6  0 8 4). 3076 8 lo (^4) g (^1 4) i (^2) /o (^3) 's/second
...
这m里ys的qll>sns是el (^4) e (^2) c (^5) t (^6)  (^4) ro (^0) u (^6) n (^52) d，((最 42 近 5 一 64 次 0 检 65 查 2 点- 4 的 0 l 6 sn 8 是 4144026834 )/ 1140232 ， 4 /计 1 算 02 出 4 当)前lo日gs志iz文e_件M的B使; 用量是这两个值之差：
+|-l-o-g--s-i-z-e--_-M-+B|
+|--------- 1 - 8 - -+|
+ 1 - -r-o-w---i-n--s--e+t( 0. 00 sec)
当前日志文件的使用量大约 18 MB。


### 设置日志文件大小大考虑

MySQL默认在数据目录下有两个 48 MB的日志文件，ib_logfile 0 和ib_logfile 1 。对于繁忙的数据库，
这样的日志文件通常太小，因为当日志文件写满时，会触发检查点，把内存中的数据写入磁盘，小
的日志文件会频繁地触发检查点，增加写磁盘频率，引起系统性能下降。
大的日志文件能容纳的数据变化量大，会造成数据库在崩溃恢复时耗时较长，但新的MySQL版本的
崩溃恢复速度已经很快了，因此把日志文件设置得大一些通常不会错，甚至可以设置得和InnoDB缓
存池一样大。
另外一些备份工具要备份在备份过程中产生的重做日志， 如果日志文件过小，备份工具备份日志的
速度跟不上日志产生的速度时，需要备份的日志可能已经被覆盖了，例如XtraBackup工具可能会遇
到下面的错误：
xtrabackup: error: it looks like InnoDB log has wrapped around before xtrabackup could
process all records due to either log copying being too slow, or log files being too small.


### 计算日志产生量

一个合理大小的日志文件应该可以容纳数据库在高峰时 1 到 2 个小时的数据变化。下面的例子是查询一分钟产生的日志
量：
设置pager只显示lsn：
mPAysGqEl>R psaegt etor g'grerepp s seeqquueennccee'

查询当前的lsn：
mysql> show engine innodb status \G

L 1 o rgo (^) wse iqnu seentc (e 0 .n 0 u 0 m sbeecr) 1439955157
休眠一分钟：
mysql> select sleep( 60 );
再次查询当前的lsn：
mLoygs qsle>q suheonwce e nnugminbee irn n o d b s 1 ta 4 t 5 u 5 s 0 \ 0 G 7613
1 row in set ( 0. 01 sec)
取消设置的pager：
mysql> nopager
PAGER set to stdout
根据一分钟的采样，可以计算出一个小时产生的日志
量：
m 14 y 3 s 9 q 9 l> 5 5 s 1 e 5 le 7 c)t* 6 r 0 o/u 1 n 0 d 2 ( 4 ( (^1) / 14052540 ) (^0 7) " 16 1 h (^3) o-ur log(MB)";
+----------------+
| 1 hourlog(MB)|
+----------------+
| 861 |
+----------------+
1 row in set ( 0. 00 sec)
这里一个小时的日志量是 861 MB。


### 决定日志文件的两个参数

日志文件的大小有两个参数决定：
（ 1 ）innodb_log_files_in_group：表示一个组里有多少个文件，默认为 2 。
（ 2 ）innodb_log_file_size：表示单个日志文件的大小，默认为 48 MB。
因此如果保持innodb_log_files_in_group为 2 不变，把innodb_log_file_size设置为 860 MB，可
以容纳高峰期两个小时的日志。


### 修改日志文件大小的方法

修改日志文件大小的方法很简单，只需要修改参数文件中的innodb_log_file_size的设置，然后重新启动MySQL即可。不需
要删除当前的日志文件，在启动过程中，MySQL会发现参数值和当前日志文件的大小不一样，然后自动删除旧的日志文件，
并创建新的日志文件，在MySQL的错误日志里会有如下记录：
1 [Note] [MY- 013041 ] [InnoDB] Resizing redo log from 2 * 50331648 to 2 * 901775360 bytes, LSN= 1457612031
1 [Note] [MY- 013084 ] [InnoDB] Log background threads are being closed...
1 [Note] [MY- 012968 ] [InnoDB] Starting to delete and rewrite log files.
1 [Note] [MY- 013575 ] [InnoDB] Creating log file ./ib_logfile 101
1 [Note] [MY- 013575 ] [InnoDB] Creating log file ./ib_logfile 1
1 [Note] [MY- 012892 ] [InnoDB] Renaming log file ./ib_logfile 101 to ./ib_logfile 0
1 [Note] [MY- 012893 ] [InnoDB] New log files created, LSN= 1457612300
1 [Note] [MY- 013083 ] [InnoDB] Log background threads are being started...


### 实例优化和参数设置

数据库优化的重要性
设置系统参数
内存的分配
InnoDB日志的配置
innodb_dedicated_server参数
硬盘读写参数
资源组


### 参数 innodb_dedicated_server

MySQL 8 中新引进了一个参数innodb_dedicated_server，这个参数的默认值是off，就像这个参
数名所建议的一样，当MySQL独占当前服务器资源的时候，可以把这个参数设置为on，这时
MySQL会自动探测当前服务器的内存大小并设置下面 4 个参数：
（ 1 ）innodb_buffer_pool_size
（ 2 ）innodb_log_file_size
（ 3 ）innodb_log_files_in_group
（ 4 ）innodb_flush_method
其中前面 3 个参数是根据当前服务器的内存大小计算出来的，这样对运维在虚拟机或云上运行的
MySQL很方便，当调整了内存的大小后，MySQL会在启动时自动调整这 3 个参数，省去了每次手工
修改参数的工作。


#### innodb_buffer_pool_size 根据物理内存的设置策略

内存大小    innodb_buffer_pool_size的值

小于1GB     128 MB

1GB到4GB    物理内存× 0. 5

大于4GB     物理内存× 0. 75


### innodb_log_file_size 设置策略

innodb_log_file_size和innodb_log_files_in_group两个参数是根据innodb_buffer_pool_size计
算出来的。

```
innodb_buffer_pool_size innodb_log_file_size
小于 8 GB                512 MB
8 GB到 16 GB             1024 MB
大于 16 GB               2 GB
```

### innodb_log_file_in_group 设置策略

innodb_buffer_pool_size innodb_log_files_in_group

```
小于 8 GB              以GB为单位对innodb_buffer_pool_size取整
8 GB到 128 GB          以GB为单位对（innodb_buffer_pool_size* 0. 75 ）取整
大于 128 GB             64
```

### 显式设置的参数优先生效

当参数innodb_dedicated_server为ON时，如果还显式设置了这些参数，则显式设置的这些参数
会优先生效，并且在MySQL的错误日志中会记录如下内容：

0 [Warning] [MY- 012360 ] [InnoDB] Option innodb_dedicated_server is ignored for
innodb_log_file_size because innodb_log_file_size= 2073034752 is specified explicitly.
显式指定某一个值，并不会影响另外 3 个参数值的自动设定。


### MySQL 的启动探测

当参数innodb_dedicated_server为ON时，MySQL每次启动时会自动探测服务器的内存并自动调
整上述几个参数值。在任何时候MySQL都不会将自适应值保存在持久配置中，利用这个参数就可以
保证服务器（包括虚拟机或者容器）扩展以后，MySQL能“自动适应”，以尽量利用更多的服务器
资源


### 实例优化和参数设置

数据库优化的重要性
设置系统参数
内存的分配
InnoDB日志的配置
innodb_dedicated_server参数
硬盘读写参数
其他参数
资源组


### 硬盘读写参数

硬盘的读写通常是对数据库性能最大的因素之一。这里介绍几个影响硬盘读写的重要参数。

innodb_flush_log_trx_commit
sync_binlog
innodb_flush_method
innodb_io_capacity和innodb_io_capacity_max


### innodb_flush_log_at_trx_commit

innodb_flush_log_at_trx_commit参数控制事务提交时写重做日志的行为方式，它有三个值： 0 、 1 和 2 。
（ 1 ）默认值为 1 ，每次事务提交的时候都会将日志缓存中的数据写入到日志文件，同时还会触发文件系统到磁
盘的同步，如果发生系统崩溃，数据是零丢失，这种方式对数据是最安全的，但性能是最慢的，因为把数据从
缓存同步到磁盘的成本很高。这种方式适用于对数据安全性要求高的行业，如银行业。但很多互联网的应用，
对数据的安全性要求不太高，而对性能的要求很高，设置成 0 或 2 会更合适。
（ 2 ）设置成 0 时，事务提交的时候不会触发写日志文件的操作，日志缓存中的数据以每秒一次的频率写入到日
志文件中，同时还会进行文件系统到磁盘的同步操作。
（ 3 ）设置成 2 时，事务提交的时候会写日志文件，但文件系统到磁盘的同步是每秒进行一次。
0 和 2 都是每秒进行一次文件系统到磁盘的同步，因此这两种方式的性能都差不多，当系统崩溃时，最多丢失 1
秒的数据。但 0 和 2 还有细微的不同，当设置成 2 时，每次事务提交都写日志文件，因此数据已经从MySQL的日
志缓存刷新到了操作系统的文件缓存，如果只是MySQL崩溃，而操作系统没有崩溃，将不会丢失数据。因此 0
和 2 比较起来，通常设置为 2 比较好。


### sync_binlog

sync_binlog参数控制事务提交时写二进制日志的行为方式，它有三个值： 0 、 1 和N。
（ 1 ）默认值为 1 ，每次事务提交的时候都会把二进制日志刷新到磁盘，这种方式对数据是最安全的，
但性能是最慢的。
（ 2 ）设置成 0 时，事务提交的时候不会把二进制日志刷新到磁盘，刷磁盘的动作由操作系统控制。
（ 3 ）设置成N（N不等于 0 或 1 ）时，每进行N事务提交后会进行一次把二进制日志刷新到磁盘的动
作。
没有备库和使用二进制日志进行时间点恢复的需求时，可以把sync_binlog参数设置为 0 或N，设置
为 0 是把刷新二进制日志文件的操作交给操作系统决定，但操作系统可能会在二进制日志文件写满
进行切换时才刷新磁盘文件，这样会造成数秒的延迟，在这期间事务无法提交，因此把这个参数设
置成 100 或 1000 之类的一个合理数值比设置成 0 好。


### sync_binlog

如果使用二进制日志进行主库和备库之间的数据同步，或者使用二进制日志进行时间点恢复，并且
对数据一致性要求高时，把sync_binlog参数设置为 1 ，同时要把innodb_flush_log_trx_commit参
数也设置为 1 。把这两个参数都设置成 1 对性能的负面影响很大，为了提高性能，这时使用的存储应
该是带缓存的，并且设置成Write-back，而不是Write-through，这样数据只写入到存储的缓存中
即返回。但存储的缓存应该是带电池的，如果缓存不带电池，或者电池没有电，突然发生掉电的时
候，不仅数据会丢失，而且会造成数据库损坏，无法启动，这种情况要比丢失一秒钟的数据要糟糕
得多。
写二进制日志的成本比写重做日志的成本要高得多，因为重做日志的大小和文件名是固定的，重做
日志循环写入日志文件。而每次写二进制日志时，文件都会进行扩展，如果写满了还要新建文件，
这样每次写二进制日志不但要写数据，还要修改二进制日志文件的元数据，因此把sync_binlog设
置成 1 比把innodb_flush_log_trx_commit设置成 1 对性能负面影响还要大得多。


### innodb_flush_method

innodb_flush_method参数控制MySQL将数据刷到InnoDB的数据文件和日志文件的动作。在
Windows系统上有两个选项：unbuffered是默认和推荐的选项，另外一个是normal。Linux系统上，
常用的选项有一下几种：
1. fsync：是默认值，使用fsync()系统调用刷新数据文件和日志文件，数据会在操作系统的缓存
中保存。
2. O_DSYNC：InnoDB使用O_SYNC打开和刷新日志文件，使用fsync()刷新数据文件。
3. O_DIRECT：使用O_DIRECT打开数据文件，使用fsync()系统调用刷新数据文件和日志文件，
数据不会在操作系统的缓存中保存。
4. O_DIRECT_NO_FSYNC：使用O_DIRECT刷新I/O，但写磁盘时不执行fsync()。


### innodb_flush_method

通常对于硬盘性能好的服务器，可以设置成O_DIRECT，这样避免在InnoDB缓存和操作系统缓存
中存有两份数据，而且InnoDB缓存比操作系统缓存效率要高，因为InnoDB缓存是专门针为
InnoDB的数据设计的，而操作系统缓存是为通用的数据设计的。
设置成O_DIRECT_NO_FSYNC时，因为写磁盘时不执行fsync()，速度可能会快，但突然断电时
可能会丢失数据。
对于读操作大大多于写操作的应用，设置成fsync会比设置成O_DIRECT性能略好。
但如何选择这些参数最终需要经过测试才能确定，测试时要注意观察状态参数
Innodb_data_fsyncs，它记录着调用fsync()的次数。通常fsync和O_DIRECT调用fsync()的次数
差不多，O_DIRECT_NO_FSYNC调用fsync()的次数最少。


#### innodb_io_capacity 和 innodb_io_capacity_max

InnoDB后台线程会进行一些I/O操作，例如把缓冲池中的脏页刷新到磁盘，或将更改从更改缓冲区
写入到对应的二级索引。InnoDB试图以不影响服务器正常工作的方式执行这些I/O操作，这需要它
知道系统的I/O的处理能力，它根据参数innodb_io_capacity评估系统的I/O带宽。参数
innodb_io_capacity_max值定义了系统I/O能力的上限，防止在I/O的峰值时消耗服务器的全部I/O
带宽。
通常可以把innodb_io_capacity设置得低一些，但不要低到后台I/O滞后的程度。如果该值太高，
数据将很快从缓冲池中被移除，不能充分发挥缓存的优势。但对于繁忙而且具有较高I/O处理能力的
系统，可以设置一个较高的值来帮助服务器处理与数据快速变更相关联的后台维护工作。


#### innodb_io_capacity 和 innodb_io_capacity_max

这两个参数的默认值如下：
mysql>showvariableslike'innodb_io_capacity%';
+------------------------+-------+
|Variable_name|Value|
+------------------------+-------+
|innodb_io_capacity| 200 |
|innodb_io_capacity_max| 2000 |
+------------------------+-------+
这两个参数的设定是基于系统的每秒能处理的I/O数量（IOPS），可以把innodb_io_capacity_max设置成极
限的IOPS，innodb_io_capacity设置成它的一半左右。目前业界有很多I/O测试软件可以测出系统的IOPS，
也可以通过硬盘配置进行估算，例如一块 15 K转速的传统硬盘的IOPS的参考值大约是 200 ，高端SSD盘可以达
到 60 万。
状态参数Innodb_data_fsyncs记录着数据刷新到磁盘的次数，把innodb_io_capacity调大后，可以看到这个
状态参数也相应的增加了。


### 实例优化和参数设置

数据库优化的重要性
设置系统参数
内存的分配
InnoDB日志的配置
innodb_dedicated_server参数
硬盘读写参数
其他参数
资源组


### 其他参数

max_connections
skip_name_resolve
binlog_order_commits


### max_connections

系统参数max_connections设置了允许的服务器最多连接数，防止服务器因为连接数过多而造成资
源耗尽，默认是 151 ，这在生产环境通常偏小。这个参数应当设置为经过压力测试验证后系统能承
受的最多连接数。可以参考状态参数Max_used_connections和Max_used_connections_time，
它们记录了系统连接数曾经达到的最大值和发生时间。
mysql> show status like 'max_used%';


### binlog_order_commits

系统参数binlog_order_commits默认为on，如果把这个参数设置为off将不能保证事务的提交顺序
和写入二进制日志的顺序一致，这不会影响到数据一致性，在高并发场景下还能提升一定的吞吐量。


### skip_name_resolve

系统参数skip_name_resolve默认为off，这时MySQL每收到一个连接请求，都会进行正向和反向
DNS解析，建议设置成on，禁止域名解析，这样会加快客户端连接到MySQL服务器的速度。当
DNS服务器运行正常时，这个优势并不明显，如果DNS服务器出故障，或者变慢，进行域名解析的
时间可能会很长，甚至会拒绝连接。如果解析不成功，在错误日志里面会有类似下面的提示：
40162 [Warning] [MY- 010055 ] [Server] IP address ' 192. 168. 87. 178 ' could not be resolved:
Name or service not known
把这个参数设置成on也有弊端，就是只能使用IP进行grant赋权，不能使用主机名，通常主机名不
会变，而IP改变的可能性比主机名大。因此在一个生产主机上把skip_name_resolve从off改成on
要小心，因为原来用主机名赋予的权限不能用了。


### 实例优化和参数设置

数据库优化的重要性
设置系统参数
内存的分配
InnoDB日志的配置
innodb_dedicated_server参数
硬盘读写参数
其他参数
资源组


### 资源组

MySQL 8 中引入了资源组（Resource Groups）的概念，它可以设定某一类SQL语句所允许使用
的资源（目前只包括CPU）。在高并发的系统中，资源组可以保证关键交易的性能，例如可以设定
市场统计类的交易在白天使用较少的资源，以免影响客户的交易，在晚上可以使用较多的资源。


### 查询资源组

在information_schema.resource_groups视图中可以查询资源中的信息，默认有两个资源组：
mysql> select * from information_schema.resource_groups\G
*************************** 1. row ***************************
RESOURCE_GROUP_NAME: USR_default
RESOURCE_GROUP_TYPE: USER
RESOURCE_GROUP_ENABLED: 1
VCPU_IDS: 0 - 3
THREAD_PRIORITY: 0
*************************** 2. row ***************************
RESOURCE_GROUP_NAME: SYS_default
RESOURCE_GROUP_TYPE: SYSTEM
RESOURCE_GROUP_ENABLED: 1
VCPU_IDS: 0 - 3
THREAD_PRIORITY: 0
2 rows in set ( 0. 03 sec)


### 创建资源组

使用create resource group语句可以创建资源组，创建一个batch用户资源组的例子如下：
mysql> create resource group Batch type = user vcpu = 2 - 3 thread_priority = 10 ;
创建完成后查看这个用户资源组的信息如下：
mysql> select * from information_schema.resource_groups where resource_group_name =
'Batch'\G
*************************** 1. row ***************************
RESOURCE_GROUP_NAME: Batch
RESOURCE_GROUP_TYPE: USER
RESOURCE_GROUP_ENABLED: 1
VCPU_IDS: 2 - 3
THREAD_PRIORITY: 10


### 修改资源组的属性

在系统高负载的时间段，减少分配给资源组的CPU数量，并降低其优先级：

mysql> alter resource group Batch vcpu = 3 thread_priority = 19 ;
在系统负载较轻的情况下，增加分配给组的CPU数量，并提高其优先级：

mysql> alter resource group Batch vcpu = 0 - 3 thread_priority = 0 ;
注意，用户线程的优先级不能小于 0 ：
mysql> alter resource group Batch vcpu = 3 thread_priority = - 9 ;
ERROR 3654 (HY 000 ): Invalid thread priority value - 9 for User resource group Batch.
Allowed range is [ 0 , 19 ].


### 使用资源组

激活Batch资源组的命令如下：
mysql> alter resource group Batch enable;
删除Batch资源组的命令如下：
mysql> drop resource group Batch;
要将线程分配给Batch资源组，执行以下操作：
mysql> set resource group Batch for thread_id;
当thread_id有多个时，中间用逗号隔开。
如果要把当前线程设定到 Batch资源组中，在会话中执行以下语句：
mysql> set resource group batch;
此后，会话中的语句将使用Batch资源组的资源进行执行 。
要使用Batch组执行单个语句 ，请使用 resource_group优化程序提示：
mysql> insert /*+ resource_group(Batch) */ into t 2 values( 2 );
在SQL语句里设置提示的方法可以和MySQL的中间件结合起来使用，例如ProxySQL支持在SQL语句中增加提示。


### 查询线程使用的资源组

可以在performance_schema.threads视图中的resource_group字段查询线程使用的资源组，相
应的命令和输出结果如下：

mysql> select thread_id, resource_group from performance_schema.threads where
thread_id= 10054 ;
+-----------+----------------+
|thread_id|resource_group|
+-----------+----------------+
| 10054 |Batch|
+-----------+----------------+
1 row in set ( 0. 00 sec)


### 资源组的限制

资源组目前在使用中还是一些限制：
1. 如果安装了线程池插件，则资源组不可用。
2. 资源组在macOS上不可用，因为它不提供用于将CPU绑定到线程的API。
3. 在FreeBSD和Solaris上，忽略资源组线程优先级，尝试更改优先级会导致警告。实际上，所有
线程都以优先级 0 运行。
4. 在Linux上，需要对mysqld进程设置CAP_SYS_NICE功能，否则将忽略资源组线程优先级。


### 在 Linux 上设置 CAP_SYS_NICE 功能

CAP_SYS_NICE可以使用setcap命令手动设置该功能，使用getcap检查功能。相应命令和输出结
果如下：

# setcap cap_sys_nice+ep /usr/sbin/mysqld
# getcap /usr/sbin/mysqld
/usr/sbin/mysqld = cap_sys_nice+ep
或者使用sudo systemctl edit mysql在MySQL服务里增加下面的内容：
[Service] AmbientCapabilities=CAP_SYS_NICE
然后重新启动MySQL服务，设置线程优先级才能生效。


### Windows 平台上的线程的优先级

```
线程优先级范围 Windows优先级
```
- 20 到- (^10) THREAD_PRIORITY_HIGHEST

- 9 到- 1 THREAD_PRIORITY_ABOVE_NORMAL
    0 THREAD_PRIORITY_NORMAL
1 到 10 THREAD_PRIORITY_BELOW_NORMAL

10 到 (^19) THREAD_PRIORITY_LOWEST


### 实例优化和参数设置课程回顾

数据库优化的重要性
设置系统参数
内存的分配
InnoDB日志的配置
innodb_dedicated_server参数
硬盘读写参数
其他参数
资源组


### 最新课程请联系我


## 姚远

### MySQL 8 优化 —— 找出 TOP SQL


### 找出 TOP SQL

当要对MySQL进行优化时，找到TOP SQL语句通常是第一步。这里介绍的使用不需要另外安装工
具的找出需要优化的SQL的方法。
● 从操作系统层监控到的最繁忙线程找出TOP SQL
● 慢查询日志
● 性能视图
● sys数据库中的存储过程
● diagnostics()存储过程
● ps_trace_statement_digest()存储过程
● statement_performance_analyzer()存储过程
● ps_trace_thread()存储过程


### 找出 TOP SQL

● 从操作系统层监控到的最繁忙线程找出TOP SQL
● 慢查询日志
● 性能视图
● sys数据库中的存储过程
● diagnostics()存储过程
● ps_trace_statement_digest()存储过程
● statement_performance_analyzer()存储过程
● ps_trace_thread()存储过程


### 繁忙的线程执行着 TOP SQL

例如使用linux系统中的top加上-H参数可以按查看按繁忙程度排序的线程：

PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND

29345 mysql 20 0 2828020 512200 18624 R 22. 9 13. 2 0 : 24. 10 mysqld

26089 mysql 20 0 2828020 512200 18624 D 6. 0 13. 2 1 : 57. 51 mysqld


### 繁忙的线程执行着 TOP SQL

可以看到最繁忙的线程号是 29345 ，根据线程号可以从performance_schema.threads视图中找到正在执行的SQL语句：
mysql> select * from performance_schema.threads where thread_os_id= 29345 \G
*************************** 1. row ***************************
THREAD_ID: 75
NAME: thread/sql/one_connection
TYPE: FOREGROUND
PROCESSLIST_ID: 32
PROCESSLIST_USER: root
PROCESSLIST_HOST: localhost
PROCESSLIST_DB: mysqlslap
1 row in set ( 0. 00 sec)

```
PROCESSLIST_COMMAND: Query
PROCESSLIST_TIME: 0
PROCESSLIST_STATE: waiting for handler commit
PROCESSLIST_INFO: insert into table_a(col 2 ) values(md 5 (rand()))
PARENT_THREAD_ID: NULL
ROLE: NULL
INSTRUMENTED: YES
HISTORY: YES
CONNECTION_TYPE: Socket
THREAD_OS_ID: 29345
RESOURCE_GROUP: USR_default
1 row in set ( 0. 00 sec)
```

### 繁忙的线程执行着 TOP SQL

如果有必要，可以使用PROCESSLIST_ID字段指定的线程号杀死这个线程或正在执行的SQL，不
能使用THREAD_ID字段值执行kill命令，不然可能会杀错了线程：

mysql> kill query 32 ;
或者
mysql> kill 32 ;


### 找出 TOP SQL

● 从操作系统层监控到的最繁忙线程找出TOP SQL
● 慢查询日志
● 性能视图
● sys数据库中的存储过程
● diagnostics()存储过程
● ps_trace_statement_digest()存储过程
● statement_performance_analyzer()存储过程
● ps_trace_thread()存储过程


### 慢查询日志

long_query_time：响应时间超过这个参数值的SQL语句被定义为慢SQL，默认 10 秒。
slow_query_log：是否激活慢查询日志，默认为off。
slow_query_log_file：慢速查询日志文件的路径和文件名，默认/var/lib/mysql/localhost-
slow.log。
log_slow_extra：从MySQL 8. 0. 14 版本开始才有的，当它为true时，将记录与执行的SQL语句相
关的额外信息


### 慢查询日志

下面是设置long_query_time = 0 时记录的一条SQL语句的默认内容如下：

# Time: 2021 - 01 - 22 T 16 : 22 : 21. 177507 + 08 : 00
# User@Host: root[root] @ localhost [] Id: 115
# Query_time: 0. 000781 Lock_time: 0. 000361 Rows_sent: 1 Rows_examined: 1
SET timestamp= 1611303741 ;
SELECT s_quantity, s_data, s_dist_ 01 FROM stock WHERE s_i_id = 48241 AND s_w_id =
3 ;


### 慢查询日志

当参数log_slow_extra 设置为on时，执行同样的SQL语句，记录的信息如下：
# Time: 2021 - 01 - 22 T 17 : 13 : 08. 765664 + 08 : 00
# User@Host: root[root] @ localhost [] Id: 117
# Query_time: 0. 000558 Lock_time: 0. 000330 Rows_sent: 1 Rows_examined: 1 Thread_id: 117
Errno: 0 Killed: 0 Bytes_received: 0 Bytes_sent: 265 Read_first: 0 Read_last: 0 Read_key: 1
Read_next: 0 Read_prev: 0 Read_rnd: 0 Read_rnd_next: 0 Sort_merge_passes: 0
Sort_range_count: 0 Sort_rows: 0 Sort_scan_count: 0 Created_tmp_disk_tables: 0
Created_tmp_tables: 0 Start: 2021 - 01 - 22 T 17 : 13 : 08. 765106 + 08 : 00 End: 2021 - 01 -
22 T 17 : 13 : 08. 765664 + 08 : 00
SET timestamp= 1611306788 ;
SELECT s_quantity, s_data, s_dist_ 01 FROM stock WHERE s_i_id = 48241 AND s_w_id = 3 ;


### Slow_queries 状态参数

在slow_query_log为on时，记录慢查询语句的个数。

# mysql - e "select 1 where 0 =sleep( 11 )"
+---+
| 1 |
+---+
| 1 |
+---+
# mysqladmin extended-status |grep - i slow_q
| Slow_queries | 1


### 找出 TOP SQL

● 从操作系统层监控到的最繁忙线程找出TOP SQL
● 慢查询日志
● 性能视图
● sys数据库中的存储过程
● diagnostics()存储过程
● ps_trace_statement_digest()存储过程
● statement_performance_analyzer()存储过程
● ps_trace_thread()存储过程


### 性能视图

在视图sys.statement_analysis中找出总计执行时间最长的SQL语句：

mysql> select * from sys.statement_analysis limit 1 \G
视图sys.statement_analysis已经按照总延迟时间降序排序，因此第一条记录就是总计用时最长的
SQL。还可以在视图sys.statements_with_runtimes_in_ 95 th_percentile中可以查询到运行时间
最长的 5 %的语句。


### 性能视图

下面的SQL语句列出平均执行时间最长的语句，这类SQL语句通常优化的空间最大：

mysql> select * from performance_schema.events_statements_summary_by_digest order
by avg_timer_wait desc limit 1 \G


### 性能视图

下面的SQL语句列出执行次数最多的SQL语句，这类SQL语句通常对整体系统性能影响最大：

mysql> select * from performance_schema.events_statements_summary_by_digest order
by count_star desc limit 1 \G


### 性能视图

下面的SQL语句列出检查行数最多的SQL语句，这类SQL语句通常消耗最多硬盘的读写：

mysql> select * from performance_schema.events_statements_summary_by_digest order
by sum_rows_examined desc limit 1 \G


### 性能视图

下面的SQL语句列出返回行数最多的SQL语句，这类SQL语句通常占用最多网络带宽：

mysql> select * from performance_schema.events_statements_summary_by_digest order
by sum_rows_sent desc limit 1 \G


### 性能视图

sys.statements_with_runtimes_in_ 95 th_percentile视图中包括了最慢的 5 %的SQL语句：

mysql> select * from sys.statements_with_runtimes_in_ 95 th_percentile\G


### 找出 TOP SQL

● 从操作系统层监控到的最繁忙线程找出TOP SQL
● 慢查询日志
● 性能视图
● sys数据库中的存储过程
● diagnostics()存储过程
● ps_trace_statement_digest()存储过程
● statement_performance_analyzer()存储过程
● ps_trace_thread()存储过程


### sys 存储过程

性能视图记录的信息是自系统启动以来的所有信息，当前的性能问题可能在视图里被稀释了，使用
sys的存储过程可以收集当前的信息信息。
1. diagnostics()存储过程
2. ps_trace_statement_digest()存储过程
3. statement_performance_analyzer()存储过程
4. ps_trace_thread()存储过程


### sys 存储过程

diagnostics()存储过程会生成一个关于当前MySQL实例整体性能的诊断报告，例如：

mysql> tee diagnostics.log
mysql> call sys.diagnostics(null,null,'current');
mysql> notee;


### sys 存储过程

ps_trace_statement_digest()存储过程可以根据提供的SQL语句摘要哈希值跟踪收集这些SQL语
句的执行过程中的性能诊断信息。
下面的例子是在 60 秒内跟踪指定的SQL语句，每 0. 1 秒收集一次性能诊断信息：

mysql> call sys.ps_trace_statement_digest(@digest, 60 , 0. 1 , true, true);


### sys 存储过程

statement_performance_analyzer()存储过程可以生成当前MySQL实例中正在运行的SQL语句的
两个快照，并对比这两个快照，生成增量报告。


### sys 存储过程

ps_trace_thread()存储过程可以跟踪某个线程的执行过程，把这个线程执行的所有SQL语句的性
能信息都记录下来，并输出报告。这个存储过程适合用于执行存储过程或多个SQL语句的线程。
例如：采用当前的配置启动对 51 号线程的跟踪，跟踪 60 秒，每 1 秒收集一次性能信息，生成性能报
告文件：
mysql> call sys.ps_trace_thread( 51 ,'/tmp/td_ 51 .dot',null, null, true,false, false);


### 回顾找出 TOP SQL 的方法

当要对MySQL进行优化时，找到TOP SQL语句通常是第一步。这里介绍的使用不需要另外安装工
具的找出需要优化的SQL的方法。
从操作系统层监控到的最繁忙的线程找出TOP SQL
慢查询日志
性能视图
sys数据库中的存储过程
● diagnostics()存储过程
● ps_trace_statement_digest()存储过程
● statement_performance_analyzer()存储过程
● ps_trace_thread()存储过程


### 最新课程请联系我


## 姚远

### SQL 语句的执行计划


### SQL 语句的执行计划

z SQL语句只是告诉了数据库要做什么，并没有告诉数据库如何做，查看SQL语句的执行计划可以
使SQL的执行过程从黑盒变成白盒。
z 在SQL语句前面加上EXPLAIN即可查看SQL语句的执行计划，但不会实际执行这个SQL语句。
z 显示SQL语句的执行计划的格式有三种，分别是：传统（TRADITIONAL）、JSON和树形
（TREE）格式。可以使用FORMAT=TRADITIONAL|JSON|TREE来指定格式，默认是传统格
式。
z 显示SQL语句的执行计划不光支持select，还支持delete、insert、replace和update，但
explain analyze是例外。


### 传统格式

传统格式提供了执行计划的概况、索引的使用等基本信息，下面是一个查询SQL语句的执行计划的
例子：

mysql> explain select city from city where country_id=(select country_id from country
where country='China') ；


### 解读 SQL 的执行计划

在MySQL官方文档中的第 8 章Optimization中有关于explain的详细介绍。

https://dev.mysql.com/doc/refman/ 8. 0 /en/execution-plan-information.html

执行计划中有两个ref：
type：连接类型，ALL是全表扫描，这是成本最高的访问方式，ref是使用非唯一索引访问，const
是使用主键或唯一索引访问。
ref：索引过滤的字段，const代表常量。


### 警告信息

EXPLAIN语句执行完成后提示有一个警告信息，警告信息里包括的是优化器重写的伪SQL，这个
SQL不一定是能执行的，使用\W打开\w关闭警告信息。
下面的命令显示前面的SQL执行计划的警告信息：

mysql> show warnings\G


### JSON 格式

JSON格式提供了以JSON格式显示的详细执行计划，这个格式适合于被程序调用，例如图形工具
workbench显示的图形化的执行计划就是调用了JSON格式的接口。下面是前面SQL语句的JSON
格式的执行计划：

mysql> explain format=json select city from city where country_id=(select country_id from
country where country='China') \G
*************************** 1. row ***************************
EXPLAIN: {
"query_block": {
"select_id": 1 ,
...
在JSON格式中cost_info元素提供了估算的执行成本。


### 图形方式

MySQL的图形工具MySQL Workbench可以以图形方式显示JSON格式的执行计划。在MySQL
Workbench里显示执行计划有两种方法：
（ 1 ）在SQL语句没有执行时，在SQL语句的输入框的上方有个闪电上面带放大镜的图标，点击这
个图标就会生产这个SQL语句的执行计划。这种方式适合于SQL语句执行时间长，或者SQL语句将
修改数据的情况下使用。
（ 2 ）执行SQL语句后，在输出结果的右边有个“Execution Plan”的图标，点击这个图标也会以图
形方式显示SQL语句的执行计划。


### 图形方式

在这个图形里，每个方框代表一个执行步
骤，方框的上面左边数字是估算的执行成
本，右边数据是估算的输出行数，下边是
表名和索引名，其中索引名用加粗的方式
显示。方框的颜色和相对执行成本相关，
从低到高依次是：蓝、绿、黄、橘、红。
把鼠标停留在方框上还可以显示更加详细
的信息，例如这个图里鼠标就停留在最下
面的一步。


### 树形格式

树形格式是从MySQL 8. 0. 18 开始引入格式，它提供的执行计划比传统的执行计划更详细，输出格
式是树形的，例如：
mysql> explain format=tree select city from city where country_id=(select country_id from
country where country='China') \G
*************************** 1. row ***************************
EXPLAIN: - > Filter: (city.country_id = (select # 2 )) (cost= 7. 55 rows= 53 )

- > Index lookup on city using idx_fk_country_id (country_id=(select # 2 )) (cost= 7. 55
rows= 53 )
- > Select # 2 (subquery in condition; run only once)
- > Filter: (country.country = 'China') (cost= 11. 15 rows= 11 )
- > Table scan on country (cost= 11. 15 rows= 109 )


### EXPLAIN ANALYZE

EXPLAIN ANALYZE实际上是树形的执行计划的扩展，它不但提供了执行计划，还检测并执行了
SQL语句，提供了执行过程中的实际度量，例如：
mysql> EXPLAIN ANALYZE select city from city where country_id=(select country_id from
country where country='China') \G
*************************** 1. row ***************************
EXPLAIN: - > Filter: (city.country_id = (select # 2 )) (cost= 7. 55 rows= 53 ) (actual
time= 23. 970 .. 23. 989 rows= 53 loops= 1 )

- > Index lookup on city using idx_fk_country_id (country_id=(select # 2 )) (cost= 7. 55
rows= 53 ) (actual time= 23. 967 .. 23. 981 rows= 53 loops= 1 )
- > Select # 2 (subquery in condition; run only once)
- > Filter: (country.country = 'China') (cost= 11. 15 rows= 11 ) (actual
time= 0. 450 .. 0. 515 rows= 1 loops= 1 )
- > Table scan on country (cost= 11. 15 rows= 109 ) (actual time= 0. 405 .. 0. 467
rows= 109 loops= 1 )


### explain for connection

###### 在实际工作中，如果发现一个正在执行的SQL语句耗时很长，这时想查询它

###### 的执行计划，通常的做法是使用EXPLAIN生成这个SQL语句的执行计划，

###### 但因为统计信息等原因，生成的执行计划和正在执行的执行计划可能不完全

###### 相同，更好的做法是使用explain for connection查询当前正在使用的执行

###### 计划。


### explain for connection

下面的SQL查询出当前的会话号：
mysql> select connection_id();
+-----------------+
| connection_id() |
+-----------------+
| 17 |
+-----------------+
1 row in set ( 0. 00 sec)
或者使用show processlist查询会话号。在当前的会话中执行一个慢SQL语句：
mysql> select sleep( 60 ), city from city where country_id=(select country_id from country
where country='China') \G
根据会话号在其他会话里查询正在执行的SQL语句的执行计划：
mysql> explain for connection 17 \G


### 对与非 select 语句的支持

抑制输出，并不真正执行SQL语句。

mysql> explain analyze update actor set first_name='a' where first_name='a';


## 姚远

### SQL 执行性能的评估


### SQL 执行性能的评估方法

查看SQL执行性能的最简单方法是看SQL执行完成的时间，除此之外还有：
1. 性能视图
2. 状态变量

3. Explain analyze
4. 操作系统层监控


### 执行时间对比

mysql> select * from table_a where col 1 = 999 ;
+------+----------------------------------+
| col 1 | col 2 |
+------+----------------------------------+
| 999 | efc 45 f 14 c 8 bf 7 ced 3121488 ff 7 a 70123 |
+------+----------------------------------+
1 row in set ( 0. 00 sec)

mysql> select * from table_a where col 2 ='efc 45 f 14 c 8 bf 7 ced 3121488 ff 7 a 70123 ';
+------+----------------------------------+
| col 1 | col 2 |
+------+----------------------------------+
| 999 | efc 45 f 14 c 8 bf 7 ced 3121488 ff 7 a 70123 |
+------+----------------------------------+
1 row in set ( 0. 36 se姚c远) 微信号：yaoyuanace 个人公众号：数据慧眼


### 性能视图

MySQL自带的performance_schema和sys数据库的很多性能视图记录了SQL语句的执行性能:
$ mysqlshow performance_schema|grep events_statements_
| events_statements_current |
| events_statements_histogram_by_digest |
| events_statements_histogram_global |
| events_statements_history |
| events_statements_history_long |
| events_statements_summary_by_account_by_event_name |
| events_statements_summary_by_digest |
| events_statements_summary_by_host_by_event_name |
| events_statements_summary_by_program |
| events_statements_summary_by_thread_by_event_name |
| events_statements_summary_by_user_by_event_name |
| events_statements_summary_global_by_event_name |


### 等待时间最长的 3 个 SQL 语句

下面语句找出等待时间最长的 3 个SQL语句,注意观察其中与性能相关的字段：

mscyhseqml>a (^) _ snealmecet! (^) =* (^) 'fpreormfo ermveanntcse__sstactheemmean' t s _osrduemrm bayr ys_ubmy__tdimigeers_tw (^) awiht edrees (^) c limit 3 \G
*************************** 1. row ***************************
SCHEMA_NAME: mysqlslap
(^8 1) a (^) d (^8) c (^) b (^) a (^9) e (^3) d 4 D 7 IG 20 E 5 SfT 6 b: (^7) d 87 d 577 bfed 61357053 ed 281 c 7 cbe 9 a 0 c 21 e 42 adf 45
DIGEST_TEXT: INSERT INTO `table_b` ( `col 2 ` ) VALUES ( `md 5 ` ( `rand` ( ) ) )
COUNT_STAR: 999815
SUM_TIMER_WAIT: 2934990225631000
MIN_TIMER_WAIT: 518036000
AVG_TIMER_WAIT: 2935533000
MAX_TIMER_WAIT: 83531551000
SUM_LOCK_TIME: 56119158000000
SUM_ERRORS: 20
SUM_WARNINGS: 0
SUM_ROW姚S远_A^ F^ F^ E^ 微CT信E号D：: 9 ya 9 o 9 y 8 u 5 a 3 na..c.e^ 个人公众号：数据慧眼


### I/O 性能

首先重置performance_schema.file_summary_by_event_name视图：
mysql> truncate table performance_schema.file_summary_by_event_name;
然后执行一个全表扫描的语句：
mysql> select count(*) from testdb.table_a where col 2 <>'a';
最后查询等待事件wait/io/file/innodb/innodb_data_file的性能信息，这些信息反映了全表扫描的性
能：

m(myss)q"l,> s suemle_cntu emvbeenrt__onfa_mbyet,e cso_urenat_dr/e 1 a 0 d 2 , 4 a/ 1 v 0 g 2 _ 4 ti m"MeBr_ rReeaadd/ (^1) " 0 fr (^0) o (^0) m 0 00000. 0 "Avg Read Time
peeverfnotr_mnaanmcee=_'swcahiet/mioa/f.ifliele/i_nsnuomdmb/ainrny_obdyb__edvaetan_t_finlea'm\Ge where
*************************** 1. row ***************************
event_name: wait/io/file/innodb/innodb_data_file
count_read: 4368
Avg Read Time (ms): 0. 5880
MB Read: 68. 25000
1 row in set ( 0. 00 sec)


### 状态变量

MySQL的自带了 479 个状态变量（MySQL 8. 0. 22 ）用以反映MySQL的运行状态，在mysql客户端
里可以使用下面的命令查询会话和全局的状态变量：

mysql> show session status;
mysql> show global status;

关于具体状态变量的说明，参见：https://dev.mysql.com/doc/refman/ 8. 0 /en/server-status-
variables.html。


### 使用 mysqladmin 监控性能

也可以使用mysqladmin extended-status查询全局的状态变量。如果要查看一段时间状态变量的
变化情况，可以使用下面的命令：

$ mysqladmin extended-status - ri 60 - c 3 |tee my_status
其中：-i 60 表示每 60 秒重复执行一次，-r表示显示相邻两次查询的差值，-c 3 表示重复查询 3 次，tee
命令表示把输出结果同时保存到文件。


### SQL 语句的计数器

Com_XXX是SQL语句的计数器，其中Com是command的缩写，XXX是指的SQL语句类型，这些计数器包括：
Com_begin
Com_commit
Com_delete
Com_insert
Com_select
Com_update
查询这些SQL语句的计数器可以了解当前实例执行SQL语句的情况，可以使用下面的命令查询这些计数器：
$ mysqladmin extended-status | grep Com_ |grep - E 'begin|commit|delete|insert|select|update'
查询这些计数器在 10 秒间隔的变化值：
$ mysqladmin extended-status - ri 10 - c 9 | grep Com_ |grep - E
'begin|commit|delete|insert|select|update'


### 处理 InnoDB 表的行数的计数器

Innodb_rows_XXX是对应SQL语句处理InnoDB表的行数的计数器，XXX是指的SQL语句类型

Innodb_rows_deleted
Innodb_rows_inserted
Innodb_rows_read
Innodb_rows_updated
查询这些行数的计数器可以了解当前实例处理行数的情况，可以使用下面的命令查询这些计数器：
$ mysqladmin extended-status | grep Innodb_rows
查询这些计数器在 10 秒间隔的变化值：
$ mysqladmin extended-status - ri 10 - c 9 | grep Innodb_rows


### 查询单个 SQL 的状态参数

Handler_*计数器统计了句柄操作，句柄API是MySQL和存储引擎之间的接口，其中
Handler_read_*对调试SQL语句的性能很有用，在执行SQL语句之前，可以先使用flush status将
当前会话的状态变量重置为零：

mysql> flush status;
然后执行一条SQL语句：
mysql> select * from table_a where col 1 = 999 ;
再查询状态变量Handler_read_*的值：
mysql> show session status like 'Handler_read%';
对比另外一个SQL语句：
mysql> select * from table_a where col 2 ='efc 45 f 14 c 8 bf 7 ced 3121488 ff 7 a 70123 ';


### last_query_cost 状态变量

查询最后执行的SQL语句的估算成本（注意不是实际执行成本，MySQL里没有实际执行成本）：
mysql> show status like 'last_query_cost';
+-----------------+----------+
| Variable_name | Value |
+-----------------+----------+
| Last_query_cost | 1. 000000 |
+-----------------+----------+
1 row in set ( 0. 00 sec)
观察状态变量值，可以看到最后一个SQL语句执行了一次索引访问，估计执行成本是 1 。


### 如果要查询其他会话的状态变量值

可以查询视图performance_schema.status_by_thread，例如下面的SQL查询所有会话中的状态
变量Handler_write：
mysql> select * from performance_schema.status_by_thread where
variable_name='Handler_write';
+-----------+---------------+----------------+
| THREAD_ID | VARIABLE_NAME | VARIABLE_VALUE |
+-----------+---------------+----------------+
| 60 | Handler_write | 94 |
| 67 | Handler_write | 477 |-- 插入记录最多的线程
| 69 | Handler_write | 101 |
+-----------+---------------+----------------+

2 rows in set ( 0 .姚 (^01) 远 s e c ) 微信号：yaoyuanace 个人公众号：数据慧眼


### 慢查询日志中的状态变量

打开查询慢查询日志后，如果同时将系统参数log_slow_extra设置为true，也会记录和慢SQL语句
相关的状态变量。


### Explain analyze

使用explain analyze既可以看到估计成本，也能看到实际执行用时和访问的行数，而且每一步都可
以看到这些计量信息，这样对精确定位SQL语句执行瓶颈很有帮助。


### 操作系统层监控

从操作系统层也可以监控SQL语句的执行性能，MySQL需要消耗操作系统的 4 种资源：CPU、磁盘、
内存和网络，在Linux系统上可以采用监控工具包括：top、free、vmstat、iostat、mpstat、sar
和netstat等。


### SQL 执行性能的评估方法回顾

查看SQL执行性能的最简单方法是看SQL执行完成的时间，除此之外还有：
1. 性能视图
2. 状态变量

3. Explain analyze
4. 操作系统层监控


## 姚远

### 解密 MySQL 的优化器


### MySQL 的优化器介绍

MySQL的优化器负责生成SQL语句的执行计划。本节内容：

- 优化器开关
- 计算执行计划的成本
- 优化器跟踪


### 优化器开关

系统变量optimizer_switch可以控制优化器的行为，它的值是一组标志，每个标志有on或者off两个
值，以指示相应的优化器行为是否被启用或禁用。使用下面的命令查看当前优化器的值：

mysql> select @@optimizer_switch\G
*************************** 1. row ***************************

@ind@eoxp_tmimeirzgeer=_osnw,iitncdhe: (^) x_merge_union=on,index_merge_sort_union=on,index_merge_intersection=on,engine_co
neddi_tkioeny__paucschedsosw=onf=f,omna,itnedreiaxl_izcaotniodnit=ioonn_,speumshijdooinw=no=no,nlo,mosrer=scoann,m=orrn_,cfiorsstt_mbaatscehd==oonn,d,bulpolcicka_tneewseteedd_ouloto=po=no,snu,bbqatucehr
y_materialization_cost_based=on,use_index_extensions=on,condition_fanout_filter=on,derived_merge=on,use
_aipnhv_isoipbtliem_iiznedre=xoefsf,=doefrfi,vsekdip__csocnadnit=ioonn_,hpausshh_djoowinn==oonn,subquery_to_derived=off,prefer_ordering_index=on,hypergr
1 row in set ( 0. 00 sec)
官方文档： 8. 9. 2 Switchable Optimizations
（https://dev.mysql.com/doc/refman/ 8. 0 /en/switchable-optimizations.html）


### 改变优化器的开关可以控制优化器的行为

下面这个SQL语句是取两个索引的交集进行数据访问：

mysql> explain select * from actor where first_name='NICK' and
last_name='WAHLBERG'\G
...
修改optimizer_switch禁止使用index_merge_intersection的命令如下：
mysql> SET optimizer_switch='index_merge_intersection=off';
然后执行同样的SQL语句，它的执行计划就变了，不再使用两个索引的交集进行数据访问：
mysql> explain select * from actor where first_name='NICK' and
last_name='WAHLBERG'\G


### 计算执行计划的成本

为了计算执行计划的成本，优化器使用了一个成本计算模型进行成本计算。MySQL把各种操作的估
计成本放在mysql数据库的两个表中：
1. engine_cost表用于保存与特定引擎相关的操作成本，不同的引擎执行这些操作的成本不同。
2. server_cost表用于保存与服务相关的操作的成本，不同的服务器执行这些操作的成本不同，但
跟引擎没有关系。


### mysql.engine_cost

mysql> select engine_name,cost_name,cost_value,default_value from mysql.engine_cost;
+-------------+------------------------+------------+---------------+
| engine_name | cost_name | cost_value | default_value |
+-------------+------------------------+------------+---------------+
| default | io_block_read_cost | NULL | 1 |
| default | memory_block_read_cost | NULL | 0. 25 |
+-------------+------------------------+------------+---------------+


### mysql.server_cost

mysql> select cost_name,cost_value,default_value from mysql.server_cost;
+------------------------------+------------+---------------+
| cost_name | cost_value | default_value |
+------------------------------+------------+---------------+
| disk_temptable_create_cost | NULL | 20 |
| disk_temptable_row_cost | NULL | 0. 5 |
| key_compare_cost | NULL | 0. 05 |
| memory_temptable_create_cost | NULL | 1 |
| memory_temptable_row_cost | NULL | 0. 1 |
| row_evaluate_cost | NULL | 0. 1 |
+------------------------------+------------+---------------+
6 rows in set ( 0. 00 sec)


### 修改成本计算值

下面的例子是向mysql.engine_cost插入InnoDB引擎的成本计量，因为这个MySQL是安装在虚拟
机上，硬盘IO很慢：

mysql> insert into mysql.engine_cost (engine_name, device_type, cost_name,cost_value,
comment) values ('InnoDB', 0 , 'io_block_read_cost', 2 , 'Disk on virtual machine');
Query OK, 1 row affected ( 0. 06 sec)
使用下面到命令把修改刷新到内存中：
mysql> flush optimizer_costs;


### 修改成本计算值

如果把MySQL的临时表目录innodb_temp_tablespaces_dir和临时文件目录tmpdir设置到内存中
（例如：设备/dev/shm或者文件系统tmpfs），可以提高对临时表和临时文件的处理速度，例如提
高了 10 倍的处理速度，对应修改disk_temptable_create_cost和disk_temptable_row_cost成本
如下：
mysql> update mysql.server_cost set cost_value = 2 ,Comment = 'Temporary tables on
memory' where cost_name = 'disk_temptable_create_cost';
mysql> update mysql.server_cost set cost_value = 0. 05 ,Comment = 'Stored on memory
disk' where cost_name = 'disk_temptable_row_cost';
使用下面到命令把修改刷新到内存中：
mysql> flush optimizer_costs;


### 优化器跟踪

优化器跟踪（optimizer trace）功能可以跟踪优化器生成执行计划的过程，准确地知道优化器选择
执行路径的原因，使用优化器跟踪分 4 步：
（ 1 ）打开优化器跟踪功能：SET optimizer_trace="enabled=on"。
（ 2 ）执行需要跟踪的SQL语句。
（ 3 ）查询视图information_schema.optimizer_trace，重点关注trace字段中以JSON格式记录的
优化器跟踪信息。
（ 4 ）关闭优化器跟踪功能：SET optimizer_trace="enabled=off"。
如果需要跟踪多个SQL语句的优化过程，可以重复第 2 和第 3 步


### 两种执行计划的对比

这里举一个例子，先查看下面两个SQL语句的执行计划：

mysql> explain select * from payment where customer_id< 150 \G

mysql> explain select * from payment where customer_id< 200 \G

从执行计划里可以看到，当查询payment表中的客户号小于 150 的记录时使用索引，查询客户号小
于 200 的记录时走全表扫描。优化器为什么要这样选择呢？


### 生成优化器跟踪文件

mysql> set optimizer_trace="enabled=on";
mysql> select * from payment where customer_id< 150 \G
mysql> select trace into outfile 'payment_ 150 ' lines terminated by '' from
information_schema.optimizer_trace;
mysql> select * from payment where customer_id< 200 \G
mysql> select trace into outfile 'payment_ 200 ' lines terminated by '' from
information_schema.optimizer_trace;
mysql> set optimizer_trace="enabled=off";


### 对比分析成本

全表扫描的成本是 1635 ，而使用idx_fk_customer_id索引扫描customer_id < 150 的成本是
1429. 3 ，索引的成本低，chosen属性值是true表示优化器选择了走索引。
而使用idx_fk_customer_id索引扫描customer_id < 200 的成本是 1896. 2 ，索引的成本高，
chosen属性值是flase表示优化器没有选择索引，而是全表扫描。
关于优化器跟踪的详细信息参见MySQL的内部文档：
https://dev.mysql.com/doc/internals/en/optimizer-tracing.html。


## 姚远

### 使用 hint 改变执行计划


### 提示（ hint ）的作用

优化器开关会对全局或者当前会话的所有SQL语句起作用，而提示（hint）只用于控制单个SQL语
句的执行计划。同时使用时，提示的优先级高于优化器开关。
优化器分两类:
一类是优化器提示，用于控制优化器的行为；
另一类是索引提示，用于控制索引的使用。


### 实现系统参数 optimizer_switch 的功能

优化器提示（hint）可以在单个SQL语句中实现系统参数optimizer_switch的功能，例如禁止使用
索引的交集的功能可以使用下面的优化器提示实现：

mysql> explain select /*+ NO_INDEX_MERGE(actor) */ * from actor where
first_name='NICK' and last_name='WAHLBERG'\G

关于优化器提示的详细信息参见：https://dev.mysql.com/doc/refman/ 8. 0 /en/optimizer-
hints.html。


### 临时在一个 SQL 语句中设置系统变量的值

mysql> select /*+ SET_VAR(sort_buffer_size = 16 M) */ * from rental order by 1 , 2 , 3 , 4 ;
mysql> insert /*+ SET_VAR(foreign_key_checks=OFF) */ into tba values('aaa');


### hint 的作用范围

下面的例子是通过提示临时停止二级索引的唯一性检查，首先检查当前会话中的参数
unique_checks的值：

mysql> select @@unique_checks;
发现当前会话中的参数unique_checks的值是 1 ，在SQL语句中使用提示将unique_checks的值设
置为 0 :
mysql> SELECT /*+ SET_VAR(unique_checks=OFF) */ @@unique_checks;
再次检查当前会话中的参数unique_checks的值：
mysql> SELECT @@unique_checks;
可以看到只是在提示起作用的SQL语句中停止了二级索引的唯一性检查，之前和之后都没有影响。


### 索引提示的使用

索引提示控制优化器对索引的使用，常用类型如下：
USE INDEX：使用指定索引中的一个。
USE FORCE INDEX：和USE INDEX类似，而且尽量避免全表扫描。
IGNORE INDEX：不使用指定的索引。
索引提示的使用语法和优化器提示不同，它们直接放在指定的表名后面。
关于索引提示的详细信息参见：https://dev.mysql.com/doc/refman/ 8. 0 /en/index-hints.html。


### 使用案例

在没有使用索引提示时，优化器从两个索引中选择了rental_date：
mysql> explain select inventory_id from rental where rental_date between ' 2005 - 05 - 27 '
AND ' 2005 - 05 - 28 ' AND customer_id IN ( 433 , 274 , 319 , 909 )\G
如果可以确定使用另外一个索引效率更高的时候，可以使用USE INDEX指定使用另外一个索引：
mysql> explain select inventory_id from rental use index(idx_fk_customer_id)
where rental_date between ' 2005 - 05 - 27 ' and ' 2005 - 05 - 28 '
and customer_id IN ( 433 , 274 , 319 , 909 )\G
也可以使用IGNORE INDEX强制不使用rental_date索引：
mysql> explain select inventory_id from rental ignore index(rental_date)
where rental_date between ' 2005 - 05 - 27 ' and ' 2005 - 05 - 28 '
and customer_id IN ( 433 , 274 , 319 , 909 )\G


## 姚远

### InnoDB 的主键和二级索引


### 主键和二级索引

在InnoDB中有一种说法，一切都是索引（Everything is an index in InnoDB）。InnoDB的每个
表实际上是一个聚簇索引。这种把数据和索引存储在一起的方式相对于把数据和索引分开的方式节
约了大量的存储空间，当按聚簇索引查找时速度也更快。
InnoDB主键以外的索引称为二级索引（Secondary Indexes）。在InnoDB中，二级索引中不光包
含二级索引的字段，还包含对应的主键字段。如果主键很长，二级索引要使用更多的空间，因此主
键很短是有利的。通过二级索引访问记录，实际上进行了两次索引查找，第一次从二级索引中找到
主键，第二次用这个主键到聚簇索引中查找对应的行。


### InnoDB 的主索引和二级索引的结构图


### show index


### 非聚簇表的索引和数据是分开保存


### 系统参数 sql_require_primary_key

在绝大部分时候，显式地创建一个主键通常是正确的。从MySQL 8. 0. 13 后的版本，可以设置系统
参数sql_require_primary_key为on，强制创建有主键的索引，例如：

mysql> set sql_require_primary_key=on;
Query OK, 0 rows affected ( 0. 00 sec)
mysql> create table tab_c (col 1 char( 10 ));
ERROR 3750 (HY 000 ): Unable to create or change a table without a primary key, when
the system variable 'sql_require_primary_key' is set. Add a primary key to the table or
unset this variable to avoid this message. Note that tables without a primary key can
cause performance problems in row-based replication, so please consult your DBA before
changing this setting.


### 找出没有主键的表 1

mysql> select t.table_schema as database_name,
t.table_name
from information_schema.tables t
left join information_schema.table_constraints c
on t.table_schema = c.table_schema
and t.table_name = c.table_name
and c.constraint_type = 'PRIMARY KEY'
where c.constraint_type is null
and t.table_schema not in('mysql', 'information_schema', 'performance_schema', 'sys')
and t.table_type = 'BASE TABLE';


### 找出没有主键的表 2

mysql> select t 1 .table_schema,
t 1 .table_name
from information_schema.columns t 1
join information_schema.tables t 2
on t 1 .table_schema=t 2 .table_schema
and t 1 .table_name=t 2 .table_name
where t 1 .table_schema not in
('sys', 'mysql', 'information_schema', 'performance_schema')
and t 2 .table_type='BASE TABLE'
group by t 1 .table_schema, t 1 .table_name
having group_concat(column_key) not regexp 'PRI|UNI';


## 姚远

### 优化索引


### 优化索引的内容

- 正确使用索引
- 删除索引
- 创建索引
- 不可见索引
- 在长字符串上创建索引
- 使用索引减少锁


### 正确使用索引

对于字符型的字段，如果使用数字类型的值进行检索的时候，就不会使用索引：

mysql> explain select * from actor where last_name= 123 \G
同样的SQL语句把被检索的值改用字符类型时就会使用索引：

mysql> explain select * from actor where last_name=' 123 '\G
字段类型不同造成索引失效的更多情况是出现在表连接的时候，两个连接的字段从字段名上看起来
相同，但实际上字段的类型不同而造成无法使用索引。


### 正确使用索引

不正确地使用运算符也可能造成索引失效，例如对比下面两个SQL语句：

mysql> explain select * from actor where actor_id= 1 + 1 \G
mysql> explain select * from actor where actor_id- 1 = 1 \G
字段actor_id上是主键索引，在第一个SQL语句中，对这个字段过滤通过主键访问进行，但在第二
个SQL语句中，由于要对actor_id进行运算后再过滤，因此只能进行全表扫描后，把所有的记录进
行计算后才能进行过滤。


### 正确使用索引

两个SQL语句找出在 2005 年 6 月发生的租借电影的交易：

mysql> explain select * from rental where rental_date between ' 2005 - 06 - 01 ' and ' 2005 - 06 -
30 '\G
mysql> explain select * from rentalt where year(rental_date)= 2005 and
month(rental_date)= 6 \G
可以看到第一种SQL可以使用rental_date索引，而第二种SQL语句不能使用索引，这也是因为运
算符使用错误从而造成索引失效。


### 正确使用索引

对于创建在字符串字段上的B树索引，需要注意对最左边的字符子串进行匹配时可以使用索引，对
中间或后边的字符子串进行匹配时无法使用索引，对比一下下面两个SQL语句的执行计划中的索引
使用情况：

mysql> explain select * from actor where last_name like 'BALL%'\G
mysql> explain select * from actor where last_name like '%BALL%'\G


### 创建索引

在一个需要被检索但没有索引的字段上创建索引通常是提高SQL语句执行效率的最简单、有效的方
法。通过检查sys中的两个视图，可以找到需要创建的索引。
视图schema_tables_with_full_table_scans中包括所有没有使用高效索引的表，按扫描行数降序
进行排列，例如：
mysql> select * from sys.schema_tables_with_full_table_scans where
object_schema='sand';
检查与sbtest 1 表相关的SQL语句如下：
mysql> select * from sys.statements_with_full_table_scans where query like
'%sbtest 1 %'\G
一个SQL语句用了 5 秒钟扫描sbtest 1 表的 65 万行，如果在字段c上创建索引，这个SQL的执行效率
将大大提高。


### 删除索引

在同一个表上创建的索引并不是越多越好，索引除了占用额外的空间外，对DML语句的性能也有一
定的影响，因为对字段的修改都要修改相应字段的索引，因此不要在同一个表上创建过多的索引。
对于没有用和冗余的索引要删除，可以从视图sys.schema_unused_indexes中查询没有使用的索
引，这些索引可能是被删除的对象，查询sbtest数据库中没有使用的索引的SQL语句和输出结果如
下：
mysql> select * from sys.schema_unused_indexes where object_schema='sbtest';
查询视图schema_redundant_indexes中保存着重复的索引的SQL语句和输出结果如下：
mysql> select * from sys.schema_redundant_indexes\G
可以看到id_kc索引建立在k和c两个字段上，而k_ 2 索引建立在k字段上，因此k字段上有两个索引，
这里还给出了删除索引的SQL语句，但并不是所有的重复索引都需要删除，有些重复的索引可以提
高查询的效率，还需要保留。


### 删除索引

在sys.schema_index_statistics视图里还记录着索引被SQL语句使用的频率和延时，查询actor表
的索引的使用情况的SQL语句和输出结果如下：

mysql> select * from sys.schema_index_statistics where table_name='actor'\G
这个视图记录的索引使用情况也可以用来参考，和前面两个视图相结合，用来确定需要删除的索引。


### 不可见索引

在MySQL 8 里，可以设置索引为不可见，这样优化器就不会使用这样的索引，带来的一个显而易见
的优势是便于调试，当不能确定一个索引是否需要的时候，可以先把索引转换成不可见索引，运行
一段时间，确定这个索引的确不需要再把索引删除，如果后来发现这个索引还是有用的，可以再把
索引转换为可见索引，这样避免了成本高昂的创建、删除索引的动作。一个使用actor表的
last_name字段上的SQL语句的执行计划如下：
mysql> explain select * from actor where last_name='BALL'\G
把这个字段上的索引改成不可见索引的SQL语句如下：
mysql> alter table actor alter index idx_actor_last_name invisible;
再次检查这个SQL语句的执行计划如下：
mysql> explain select * from actor where last_name='BALL'\G
可以看到在索引被改成了不可见索引后，SQL语句将不会使用这个索引。但这个索引仍然被维护着。


### 不可见索引

可以使用show index查询索引是否可见，也可以在视图information_schema.statistics中查询索
引是否可见，查询某个表上的索引是否可见的SQL语句和输出结果如下：

mysql> select index_name, is_visible from information_schema.statistics where
table_schema='sakila' and table_name='actor';
+---------------------+------------+
| index_name | is_visible |
+---------------------+------------+
| idx_actor_last_name | NO |
| PRIMARY | YES |
+---------------------+------------+

2 rows in set ( 0 .姚 (^00) 远 s e c ) 微信号：yaoyuanace 个人公众号：数据慧眼


### 不可见索引

对于不可见索引可以把优化器的开关use_invisible_indexes设置为on（默认是off），从而让优化
器使用不可见索引。也可以使用提示SET_VAR将优化器开关use_invisible_indexes临时设置为on，
从而使用不可见索引，下面是使用提示让SQL语句使用不可见索引的执行计划：

mysql> explain select * /*+ SET_VAR(optimizer_switch = 'use_invisible_indexes=on') */
from actor where last_name='BALL'\G


### 在长字符串上创建索引

当在很长的字符串的字段上创建索引时，索引会变得很大而且低效，一个解决办法是crc 32 或md 5
函数对长字符串进行哈希计算，然后在计算的结果上创建索引。在MySQL 5. 7 以后的版本，可以创
建一个自动生成的字段，例如可以创建下面一个表：

mysql> create table website(
id int unsigned not null,
web varchar( 100 ) not null,
webcrc int unsigned generated always as (crc 32 (web)) not null,
primary key (id)
);
字段webcrc是根据字段web进行crc 32 哈希计算后生成的字段。


### 在长字符串上创建索引

向这个表中插入一条记录：
mysql> insert into website(id,web) values( 1 ,"<https://www.scutech.com>");
查询这个表中记录如下：
mysql> select * from website;
+----+---------------------------+------------+
| id | web | webcrc |
+----+---------------------------+------------+
| 1 | <https://www.scutech.com> | 3014687870 |
+----+---------------------------+------------+
1 row in set ( 0. 00 sec)
可以看到字段webcrc中自动生成了web字段的循环冗余校验值，在这个字段上创建索引，可以得到
一个占用空间少，而且高效的索引。


### 在长字符串上创建索引

在MySQL 8. 0. 13 以后的版本，可以直接创建函数索引，例如创建下面一个表：
mysql> create table website 8 (
id int unsigned not null,
web varchar( 100 ) not null,
primary key (id),
index ((crc 32 (web)))
);
在这个表中创建了一个函数索引，查询这个表上的索引的SQL和输出结果如下：
mysql> show index from website 8 \G
...
可以看到第一个索引是主键，第二个索引是函数索引。


### 在长字符串上创建索引

解决索引字段长的另一个办法是创建前缀索引（prefix index），前缀索引的创建语法中字段的写
法是：col_name(length)，前缀索引是对字符串的前面一部分创建索引，支持的数据类型包括：
char、varchar、binary和varbinary。创建前缀索引的关键是选择前缀的字符串的长度，长度越长，
索引的选择性越高，但存储的空间也越大。


### 在长字符串上创建索引

s如b下te：st 2 表中c字段是长度为 120 的字符串，查询在不同长度时索引的选择性的SQL语句和输出结果

mysql> select
count(distinct(left(c, 3 )))/count(*) sel 3 ,
count(distinct(left(c, 5 )))/count(*) sel 5 ,
count(distinct(left(c, 7 )))/count(*) sel 7 ,
count(distinct(left(c, 9 )))/count(*) sel 9 ,
count(distinct c)/count(*) selectivity
from sbtest 1 ;
+--------+--------+--------+--------+-------------+
| sel 3 | sel 5 | sel 7 | sel 9 | selectivity |
+--------+--------+--------+--------+-------------+
| 0. 0120 | 0. 6784 | 0. 9959 | 1. 0000 | 1. 0000 |
+--------+--------+--------+--------+-------------+

1 row in set ( 1. (^6) 姚 (^0) 远 s e c ) 微信号：yaoyuanace 个人公众号：数据慧眼


### 在长字符串上创建索引

可以看到在这个字段的前 7 位创建索引即可达到接近 1 的选择性，再增加这个索引的前缀位数，索引
的选择性并不会提高，下面是创建索引的命令：

mysql> alter table sbtest 2 add index (c( 7 ));


### 使用索引减少锁

InnoDB在访问记录的时候会对它进行加锁，索引可以减少InnoDB访问的记录数量，从而减少锁的
数量。如果没有使用索引进行过滤，对where条件里的字段的判断要到服务器层进行，InnoDB将对
没有过滤的全部记录加锁。例如下面的SQL语句：

mysql> begin;
mysql> select * from city where city='Shaoguan' for share;


### 使用索引减少锁

这个SQL语句对一条记录上共享锁，然后查询锁的情况如下：
mysql> select index_name, lock_type,lock_mode, count(*) from
performance_schema.data_locks group by index_name, lock_type, lock_mode;
+------------+-----------+-----------+----------+
| index_name | lock_type | lock_mode | count(*) |
+------------+-----------+-----------+----------+
| NULL | TABLE | IS | 1 |
| PRIMARY | RECORD | S | 602 |
+------------+-----------+-----------+----------+
2 rows in set ( 0. 00 sec)
发现有 602 个记录级别的锁，也就是对整个表对所有记录都上了锁。


### 使用索引减少锁

看一下这个SQL语句的执行计划：

mysql> explain select * from city where city='Shaoguan' for share\G
Extra: Using where
1 row in set, 1 warning ( 0. 01 sec)
可以看到在Extra字段中有Using where的说明，这是说使用了where条件进行记录的过滤，而这个
工作是服务器层做的


### 使用索引减少锁

在这个字段上创建索引后再看这个SQL语句的执行计划：

mysql> explain select * from city where city='Shaoguan' for share\G
...
Extra: NULL
1 row in set, 1 warning ( 0. 00 sec)
Extra字段中已经没有了Using where的说明，因为where条件里过滤工作由InnoDB存储引擎完成
了。


### 使用索引减少锁

再次查询锁的情况如下：
mysql> select index_name, lock_type,lock_mode, count(*) from
performance_schema.data_locks group by index_name, lock_type, lock_mode;
+------------+-----------+---------------+----------+
| index_name | lock_type | lock_mode | count(*) |
+------------+-----------+---------------+----------+
| NULL | TABLE | IS | 1 |
| id_city | RECORD | S | 1 |
| PRIMARY | RECORD | S,REC_NOT_GAP | 1 |
| id_city | RECORD | S,GAP | 1 |
+------------+-----------+---------------+----------+
4 rows in set ( 0. 00 sec)
发现只有 3 个记录级别的锁，大大减少了锁的竞争，因为InnoDB访问的记录减少了，锁自然也减少
了。 姚远 微信号：yaoyuanace 个人公众号：数据慧眼


### 使用索引减少锁

在实际工作中，如果发现大量的锁竞争，可以通过在适当字段上创建索引的方法减少锁。除了创建
索引外，还有两个减少锁竞争的方法，一个是减少事务的粒度，也就是尽量避免一个事务修改大量
的记录，或者持续较长的时间不提交。还有一个方法是调整事务的隔离级别。


### 优化索引的内容回顾

- 正确使用索引
- 删除索引
- 创建索引
- 不可见索引
- 在长字符串上创建索引
- 使用索引减少锁


## 姚远

### 覆盖索引使分页查询性能提高 30 倍

##### MySQL 8 性能优化


### 覆盖索引

覆盖索引（covering index）不是一种索引类型，是一种索引的访问方式，它是指一个SQL语句只
读取索引就可以获得需要的数据，不需要访问表，这样大大提高了I/O的效率，原因如下：

- 不需要访问表减少了I/O的次数。
- 索引通常比表小很多。
- 由于索引是按照键值顺序存储的（至少在一个页内是这样），对于按照键值进行范围查询时使用
    的是顺序I/O，相对于离散I/O性能大大提高。


### 覆盖索引的例子

在使用覆盖索引时，执行计划的Extra字段中有Using index的信息，下面是一个SQL语句的执行计划：
mysql>explainselectcustomer_id,inventory_id,rental_datefromrental\G
*************************** 1 .row***************************
id: 1
select_type:SIMPLE
table:rental
partitions:NULL
type:index
possible_keys:NULL
key:rental_date
key_len: 10
ref:NULL
rows: 16008
filtered: 100. 00
Extra:Usingindex
1 rowinset, 1 warning( 0. 01 sec)
这个SQL语句没有where条件，但仍然访问了rental_date索引，而且没有访问表。


### 查询 rental_date 索引的构成

mysql>selectcolumn_namefrominformation_schema.statisticswhere
index_name='rental_date';
+--------------+
|column_name|
+--------------+
|rental_date|
|inventory_id|
|customer_id|
+--------------+
3 rowsinset( 0. 01 sec)
发现这个SQL语句要查询的 3 个字段customer_id,inventory_id,rental_date都包含在这个索引中了，
因此只要访问这个姚索远引 即 可 微得信号到：所y有aoy需ua要na的ce数 据 ， 就 个没人公有众必号要：再数据访慧问眼表了。


### 增加对主键访问

由键于字二段级的索SQ引L实语质句上的都执包行含计主划键：，因此如果再加上主键，一样可以使用覆盖索引，下面是在输出字段中加上主
mysql>explainselectrental_id,customer_id,inventory_id,rental_datefromrental\G
*************************** 1 .row***************************
id: 1
select_type:SIMPLE
table:rental
partitions:NULL
type:index
possible_keys:NULL
key:rental_date
key_len: 10
ref:NULL
rows: 16008
filtered: 100. 00
Extra:Usingindex
1 rowinset, 1 warning( 0. 00 sec)
可以看到一样使用了覆盖索引。


### 需要访问的字段不在索引中不能使用覆盖索引

如果在上面的查询字段中再增加任意一个其他字段就不能使用覆盖索引了，例如下面的SQL语句将无法使用覆盖索引：
minyvseqnlt>oreyx_pidla=in 3  6 s 7 e\leGct*fromrentalwhererental_date=' 2005 - 05 - 24  22 : 53 : 30 'and
*************************** 1 .row***************************
id: 1
select_type:SIMPLE
table:rental
partitions:NULL
type:ref
possible_keys:rental_date,idx_fk_inventory_id
key:rental_date
key_len: 8
ref:const,const
rows: 1
filtered: 100. 00
Extra:NULL
1 rowinset, 1 warning( 0. 00 sec)
这个SQL的执行计划的字段Extra里没有Using index，因此没有使用覆盖索引。

```
姚远 微信号：yaoyuanace 个人公众号：数据慧眼
```

### 延迟关联

可以对这个SQL语句进行改写，先用一个可以使用覆盖索引的子查询查询出主键，再通过主键查找
相应的记录，这种方法称之为延迟关联（deferred join）：

mysql> explain select * from rental where rental_id in (select rental_id from rental where
rental_date=' 2005 - 05 - 24 22 : 53 : 30 ' and inventory_id= 367 )\G


### 分页查询

下面的SQL语句进行排序后从 1000 行开始查询 5 行，它的执行计划如下：

mysql> explain analyze select * from rental order by rental_date limit 1000 , 5 \G
*************************** 1. row ***************************
EXPLAIN: - > Limit/Offset: 5 / 1000 row(s) (cost= 1800. 70 rows= 5 ) (actual
time= 13. 501 .. 13. 502 rows= 5 loops= 1 )

- > Sort: rental.rental_date, limit input to 1005 row(s) per chunk (cost= 1800. 70
rows= 16067 ) (actual time= 13. 225 .. 13. 416 rows= 1005 loops= 1 )
- > Table scan on rental (cost= 1800. 70 rows= 16067 ) (actual time= 0. 216 .. 9. 418
rows= 16044 loops= 1 )
1 row in set ( 0. 01 sec)


### 使用延迟关联改写分页查询的 SQL

mysql> explain analyze select * from rental r 1 inner join (select rental_id from rental order by
rental_date limit 1000 , 5 ) r 2 on r 1 .rental_id=r 2 .rental_id\G
*************************** 1. row ***************************
EXPLAIN: - > Nested loop inner join (cost= 262. 62 rows= 5 ) (actual time= 0. 428 .. 0. 449 rows= 5
loops= 1 )

- > Table scan on r 2 (cost= 0. 51 .. 2. 56 rows= 5 ) (actual time= 0. 001 .. 0. 002 rows= 5 loops= 1 )
- > Materialize (cost= 8. 82 .. 10. 87 rows= 5 ) (actual time= 0. 408 .. 0. 410 rows= 5 loops= 1 )
- > Limit/Offset: 5 / 1000 row(s) (cost= 7. 80 rows= 5 ) (actual time= 0. 387 .. 0. 388 rows= 5
loops= 1 )
- > Index scan on rental using rental_date (cost= 7. 80 rows= 1005 ) (actual
time= 0. 046 .. 0. 306 rows= 1005 loops= 1 )
- > Single-row index lookup on r 1 using PRIMARY (rental_id=r 2 .rental_id) (cost= 0. 25 rows= 1 )
(actual time= 0. 007 .. 0. 007 rows= 1 loops= 5 )
1 row in set ( 0. 00 sec)


## 姚远

### 统计信息


### 统计信息的内容

1. 统计信息的作用
2. 统计信息的存放
3. 自动收集统计信息
4. 手动收集统计信息
5. 查询统计信息


### 统计信息的作用

MySQL统计信息是指数据库通过采样，统计出来的表、索引的相关信息，例如：表的记录数、聚集
索引的页数、字段值的基数（cardinality）等。MySQL在生成执行计划时，需要根据统计信息进行
估算，计算出代价最低的执行计划。统计信息由存储引擎负责，MySQL的服务器层并不保存任何统
计信息，这里描述的都是InnoDB的统计信息。
准确的表和索引的统计信息是优化器生成正确执行计划的基础，InnoDB的统计信息分两种，一种是
持久化的统计信息，将统计信息保存到表中，在MySQL重启后仍然有效；另一种是临时的统计信息，
统计信息保存在缓存中，MySQL重启后丢失，后面这种方式现在用得越来越少，这里不做介绍。


### 统计信息的存放

- 收集的表的统计信息存放在mysql数据库的innodb_table_stats表中。
- 索引的统计信息存放在mysql数据库的innodb_index_stats表中。
- 这两个表是普通表，不是视图。这两个表可以被update语句修改，但尽量不要这样做，因为这
    样通常会造成执行计划的恶化。
查询例子：
mysql> select * from mysql.innodb_table_stats where table_name='actor';
mysql> select * from mysql.innodb_index_stats where table_name='actor';


### 收集统计信息采用的隔离级别

InnoDB有四个隔离级别，分别是：
1. 未提交读（read uncommitted）
2. 提交读（read committed）
3. 可重复读（repeatable read）
4. 序列读（serializable）
收集统计信息采用的是未提交读，也就是未提交的数据（也叫脏数据）也会被统计。这背后的逻辑
是实际生产中大部分未提交的数据最终会被提交。


### 控制自动收集统计信息的参数

```
系统参数 表选项 默认值 说明
INNODB_STATS_PER
SISTENT
```
```
STATS_PERSISTEN
T
```
ON 是否把统计信息持久化

```
INNODB_STATS_AU
TO_RECALC
```
```
STATS_AUTO_REC
ALC
```
```
ON 当一个表的数据变化超过^10 %时是
否自动收集统计信息，两次统计信
息收集之间时间间隔不能少 10 秒。
```
```
INNODB_STATS_PER
SISTENT_SAMPLE_P
AGES
```
```
STATS_SAMPLE_PA
GES
```
```
20 统计索引时的抽样页数，这个值设
置得越大，收集的统计信息越准确，
但收集时消耗的资源越大。
```

### 查询系统参数

mysql> show variables like 'innodb_stat%';
+--------------------------------------+-------------+
| Variable_name | Value |
+--------------------------------------+-------------+
| innodb_stats_auto_recalc | ON |
| innodb_stats_include_delete_marked | OFF |
| innodb_stats_method | nulls_equal |
| innodb_stats_on_metadata | OFF |
| innodb_stats_persistent | ON |
| innodb_stats_persistent_sample_pages | 20 |
| innodb_stats_transient_sample_pages | 8 |
| innodb_status_output | OFF |
| innodb_status_output_locks | OFF |
+--------------------------------------+-------------+
9 rows in set ( 0. 01 sec)


### 查询表属性

```
在tab视le图 查in询for非m默at认io值n_：schema.tables的CREATE_OPTIONS字段中可以查询统计信息相关的属性，或者使用show create
mysql> alter table t 1 stats_auto_recalc= 0 ;
mysql> show create table t 1 \G
*************************** 1. row ***************************
Table: t 1
Create Table: CREATE TABLE `t 1 ` (
`intcol 1 ` int DEFAULT NULL,
`intcol 2 ` int DEFAULT NULL,
`charcol 1 ` varchar( 128 ) DEFAULT NULL,
`charcol 2 ` varchar( 128 ) DEFAULT NULL,
`charcol 3 ` varchar( 128 ) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf 8 mb 4 COLLATE=utf 8 mb 4 _ 0900 _ai_ci STATS_AUTO_RECALC= 0
1 row in set ( 0. 00 sec)
```

### 应用案例

- 当进行大批量数据导入时，可以把INNODB_STATS_AUTO_RECALC设置为OFF，避免在数
    据导入的过程中不断地收集不准确的统计信息，在数据导入完成后再手动收集统计信息并把这
    个参数设置为ON。
- 对于数据量变化大的表，例如从其他数据库导入的报告表，可以将这类表的
    STATS_AUTO_RECALC设置为OFF，等数据量稳定后再手动进行统计信息的收集。
- 对于数据分布不规则的表，可以通过增大表的STATS_SAMPLE_PAGES选项，提高收集的统
    计信息的准确性。


### 自动收集统计信息的例子

检查当前的innodb_stats_auto_recalc参数：

mysql> select @@innodb_stats_auto_recalc;
下面检查表tab_a的表统计信息：

mysql> select last_update,n_rows,clustered_index_size from mysql.innodb_table_stats
where table_name='tab_a';
下面检查表tab_a的索引统计信息：
mysql> select last_update,stat_name,stat_value,sample_size from
mysql.innodb_index_stats where table_name='tab_a';


### 自动收集统计信息的例子

把mysql客户端的提示符改成当前时间：

mysql> prompt \D>
PROMPT set to '\D> '
下面的SQL向表里面增加超过 10 %的记录：
Fri May 14 16 : 55 : 39 2021 >insert into tab_a select * from tab_a limit 2700 ;
再次检查表tab_a的表统计信息，发现已经自动进行了统计信息的收集。


### 自动收集统计信息的例子

下面的SQL把这个表的属性改成stats_auto_recalc= 0 和stats_sample_pages = 200 ：

Fri May 14 16 : 57 : 34 2021 >alter table tab_a stats_auto_recalc= 0 , stats_sample_pages =
200 ;
再次向这个表里面新增超过 10 %的记录：
Fri May 14 17 : 00 : 59 2021 >insert into tab_a select * from tab_a limit 3000 ;
Query OK, 3000 rows affected ( 0. 06 sec)
第 3 次检查表tab_a的表统计信息，发现统计信息没有发生变化，也就是不会自动进行统计信息的收
集了，因为表的属性STATS_AUTO_RECALC被设置成了 0 。


### 自动收集统计信息的例子

手工使用analyze table进行统计信息的收集：

Fri May 14 17 : 04 : 52 2021 >analyze table tab_a;

第 4 次检查表tab_a的表统计信息。发现这时的统计信息很准，因为抽样页被改成了 200
（STATS_SAMPLE_PAGES = 200 ），而这个表只有 78 页，所有的页都被抽样扫描过，因此生成
的统计信息就很精确。


### 不准确的统计信息的例子

首先把表改成不自动收集统计信息：

mysql> alter table sbtest 2 stats_auto_recalc= 0 ;
然后把表中记录增加一倍：

mysql> insert into sbtest 2 (k,c,pad) select k,c,pad from sbtest 2 ;
再检查information_schema.innodb_tablestats视图中的modified_counter字段：
mysql> select modified_counter from information_schema.innodb_tablestats where name
like 'sbtest%';
这个字段记录自上次收集统计信息后修改的行数，对比一下这个表中的实际行数：
mysql> select count(*) from sbtest 2 ;


### 不准确的统计信息的例子

再检查mysql数据库的innodb_table_stats表中记录的sbtest 2 的统计信息如下：

mysql> select last_update,n_rows,clustered_index_size from mysql.innodb_table_stats
where table_name='sbtest 2 ';
可以看到上次收集统计信息的时间距今已有一段时间了，记录的行数也只有大约真实行数的一半。
这时可以使用analyze table命令手工收集统计信息如下：
mysql> analyze table sbtest 2 ;
再重新检查mysql数据库的innodb_table_stats表中记录的sbtest 2 的统计信息如下：
mysql> select last_update,n_rows,clustered_index_size from mysql.innodb_table_stats
where table_name='sbtest 2 ';
发现收集完统计信息后，统计信息中记录的行数和真实的数据已经很接近了。


### 手工收集统计信息

手工收集统计信息有两种方法，一种是analyze table命令，它可以同时收集多个表的统计信息，例
如下面的命令收集两个表的统计信息和输出结果如下：

mysql> analyze local table actor,rental;
批量收集统计信息采用MySQL自带的工具mysqlcheck就更方便了，mysqlcheck还可以方便地被
Linux的crontab或Windows的任务管理器调用，下面的命令收集sakila数据库中所有表的统计信息
和输出结果如下：
$ mysqlcheck - -analyze sakila
也可以使用- - all-databases收集所有表的统计信息。这种情况通常在进行了批量数据导入后进行。


### 手工修改统计信息

可以通过手工修改mysql中的两个表innodb_table_stats和innodb_table_stats的对应字段来改变
统计信息，然后通过flush table把修改后到统计信息刷新到内存中，但手工修改的统计信息通常不
准，最好还是使用analyze table命令收集统计信息。


### show index 查询统计信息

一个常用的方法是的show index命令，这个命令输出的内容和视图
information_schema.statistics差不多，例如查询表sakila.actor的统计信息的SQL和输出结果如
下：

mysql> select index_name,column_name,cardinality from information_schema.statistics
where table_name='actor' and table_schema='sakila';


### show table status 查询统计信息

使用show table status可以查看相关表的统计信息，这个命令基本语法如下：

SHOW TABLE STATUS [{FROM | IN} db_name] [LIKE 'pattern' | WHERE expr]
查询actor表的命令和输出结果如下：

mysql> show table status like 'actor'\G
show table status的输出结果和视图information_schema.tables中的信息类似，在这个视图里查
询actor表的命令和输出结果如下：
mysql> select * FROM information_schema.tables where table_name='actor'\G


### 基数

基数（cardinality）是索引字段的不重复值的个数。对于主键和不包含NULL值的唯一索引，表里
的记录数就是索引的基数，因为索引中的所有值都是唯一的。表sakila.actor两个索引的基数分别
是 200 和 121 ，查询这两个索引字段的唯一值的SQL和输出结果如下：

mysql> select count(*) sum,count(distinct actor_id) 'primary _cardinality',count(distinct
last_name) 'last_name cardinality' from sakila.ator;
+-----+---------------------+-----------------------+
| sum | primary cardinality | last_name cardinality |
+-----+---------------------+-----------------------+
| 200 | 200 | 121 |
+-----+---------------------+-----------------------+


### 索引的选择性

索引的选择性（selectivity）是指基数和表中的记录总数的比值，索引的选择性越高则查询效率越
好，主键和唯一索引的选择性是 1 ，这是最高的索引选择性。一个选择性差的索引的例子是性别，
只有两个唯一值，基数是 2 ：
mysql> select count(distinct actor_id)/count(*) 'primary selectivity',count(distinct
last_name)/count(*) 'last_name selectivity'
from sakila.actor;
+---------------------+-----------------------+
| primary selectivity | last_name selectivity |
+---------------------+-----------------------+
| 1. 0000 | 0. 6050 |
+---------------------+-----------------------+


### 统计信息的内容回顾

统计信息的作用
统计信息的存放
自动收集统计信息
手动收集统计信息
查询统计信息
不准确的统计信息


## 姚远

### 直方图拯救低效率的 SQL

##### MySQL 8. 0 性能优化


### 什么是直方图统计信息？

MySQL 8 里引进直方图统计信息，用于统计字段值的分布情况。它最典型的场景是估算where子句
中过滤谓词列的选择率，以便选择合适的执行计划。直方图的适用场景包括不是索引中第一列的列、
值分布不均匀的列和在where子句中作为过滤条件的列。例如在IT公司工作的员工，男性远多于女
性，如果在性别字段上没有直方图的统计信息，优化器可能会认为男女比例各占一半，从而生成低
效率的执行计划。再如一个作业执行完成后的状态，成功的状态通常远大于失败的状态，这些信息
没有直方图统计信息的时候，MySQL的优化器无法知道。


### 没有索引也没有直方图时估计过滤比率

当需要过滤的字段上既没有索引也没有直方图时，优化器会根据MySQL代码中内置的默认规则估计
过滤的比率，实际很大程度上是瞎猜，部分常用的默认规则如下：

```
过滤类型 过滤比率（%）
= 10
<>或≠ 90
< 或 > 33. 33
between 11. 11
in 字段数× 10 和 50 的最小值
```

### 查询在 payment 表 amount 字段大于 10 的记录

mysql> explain select customer_id from payment where amount> 10 \G
*************************** 1. row ***************************
id: 1
select_type: SIMPLE
table: payment
partitions: NULL
type: ALL
possible_keys: NULL
key: NULL
key_len: NULL
ref: NULL
rows: 16086
filtered: 33. 33
Extra: Using where
1 row in set, 1 warning ( 0. 00 sec)
判断amount字段大于 10 的记录，由于这个字段上没有直方图的统计信息，优化器根据代码中内置
的默认值估计有三分之一的记录属于这个范围。


### 大于 100 的记录呢？

mysql> explain select customer_id from payment where amount> 100 \G
*************************** 1. row ***************************
id: 1
select_type: SIMPLE
table: payment
partitions: NULL
type: ALL
possible_keys: NULL
key: NULL
key_len: NULL
ref: NULL
rows: 16086
filtered: 33. 33
Extra: Using where
1 row in set, 1 warning ( 0. 00 sec)
优化器仍然估计有三分之一的记录属于这个范围，显然优化器在瞎猜。


### 在 amount 字段上创建直方图的统计信息

现在在amount字段上创建直方图的统计信息的命令和输出结果如下：

mysql> analyze table payment update histogram on amount with 256 buckets\G
*************************** 1. row ***************************
Table: sakila.payment
Op: histogram
Msg_type: status
Msg_text: Histogram statistics created for column 'amount'.
1 row in set ( 0. 31 sec)


### 有直方图时候的执行计划

mysql> explain select customer_id from payment where amount> 10 \G
*************************** 1. row ***************************
id: 1
select_type: SIMPLE
table: payment
partitions: NULL
type: ALL
possible_keys: NULL
key: NULL
key_len: NULL
ref: NULL
rows: 16086
filtered: 0. 71
Extra: Using where
1 row in set, 1 warning ( 0. 00 sec)

优化器根据直方图姚的远统 计 信 微息信号估：计y符aoy合ua这na个ce条 件 的 记 个录人公只众占号总：数数据 0 .慧 (^71) 眼%。


### 没有直方图时的执行计划

下面的SQL语句查询单词消费金额大于 10 元和在第一个店进行消费的顾客的姓名，在没有直方图时的生成的执
行计划如下：
mysql> explain analyze select first_name,last_name from customer inner join payment using
(customer_id) where amount> 10 and store_id= 1 \G
*************************** 1. row ***************************
EXPLAIN: - > Nested loop inner join (cost= 3100. 48 rows= 2918 ) (actual time= 0. 443 .. 14. 853
rows= 68 loops= 1 )

- > Index lookup on customer using idx_fk_store_id (store_id= 1 ) (cost= 36. 35 rows= 326 ) (actual
time= 0. 310 .. 0. 707 rows= 326 loops= 1 )
- > Filter: (payment.amount > 10. 00 ) (cost= 6. 72 rows= 9 ) (actual time= 0. 042 .. 0. 043 rows= 0
loops= 326 )
- > Index lookup on payment using idx_fk_customer_id (customer_id=customer.customer_id)
(cost= 6. 72 rows= 27 ) (actual time= 0. 031 .. 0. 038 rows= 27 loops= 326 )

1 row in set ( 0. 01 sec)

```
姚远 微信号：yaoyuanace 个人公众号：数据慧眼
```

### 有直方图统计信息的执行计划

mysql> explain analyze select first_name,last_name from customer inner join payment using (customer_id)
where amount> 10 and store_id= 1 \G
*************************** 1. row ***************************
EXPLAIN: - > Nested loop inner join (cost= 1672. 84 rows= 62 ) (actual time= 0. 328 .. 9. 507 rows= 68 loops= 1 )

- > Filter: (payment.amount > 10. 00 ) (cost= 1632. 85 rows= 114 ) (actual time= 0. 224 .. 8. 421 rows= 114 loops= 1 )
- > Table scan on payment (cost= 1632. 85 rows= 16086 ) (actual time= 0. 191 .. 6. 482 rows= 16049 loops= 1 )
- > Filter: (customer.store_id = 1 ) (cost= 0. 25 rows= 1 ) (actual time= 0. 009 .. 0. 009 rows= 1 loops= 114 )
- > Single-row index lookup on customer using PRIMARY (customer_id=payment.customer_id) (cost= 0. 25
rows= 1 ) (actual time= 0. 009 .. 0. 009 rows= 1 loops= 114 )

1 row in set ( 0. 02 sec)


### 两个执行计划的对比

可以看到优化器将这两个过滤条件的先后次序反转过来了，因为借助直方图统计信息，优化器知道
消费金额大于 10 元这个条件的选择性更高。从估计成本和实际执行时间都可以看出，有直方图的执
行计划效率要好很多！


### 索引和直方图的对比

索引和直方图都可以用于生成正确的执行计划，但它们之间的区别却很大：

- 索引可以用来减少访问所需行，直方图不能。当使用直方图进行查询时，它不会直接减少检查
    的行数，但可以帮助优化器选择更优化的查询计划。
- 直方图的维护成本远低于索引，对索引字段的DML操作要修改对应的索引，而直方图只在创建
    和修改的时候消耗资源。
- 索引需要占用大量的存储空间，直方图对存储空间的占用基本是零。
- 在判断某个范围内的行数时，索引的成本要高得多，因为索引需要当时使用索引试探（index
    dive）进行收集和估算，而直方图在这方面的信息是现成的。


### 桶

直方图的存放统计信息的单位是桶（bucket），默认 100 个，最多 1024 个。桶越多，收集统计信息
的时间越长，统计信息越准确。MySQL的直方图分两类：
（ 1 ）等宽(singleton)直方图：每个桶只有一个值，保存该值和累积的频率。
（ 2 ）等高(equi-height)直方图：每个桶保存上下限，累积频率以及不同值的个数。
用户不需要指定直方图的类型，MySQL会自动进行直方图类型的选择，当指定的桶数大于或等于桶
所对应的值时，创建一个等宽直方图。否则创建一个等高直方图。


### 创建一个直方图统计信息

例如在actor表的first_name字段上创建一个直方图统计信息的命令和输出结果如下：

mysql> analyze table actor update histogram on first_name\G
*************************** 1. row ***************************
Table: sakila.actor
Op: histogram
Msg_type: status
Msg_text: Histogram statistics created for column 'first_name'.
1 row in set ( 0. 01 sec)


### 删除直方图统计信息

删除这个直方图统计信息的命令和输出结果如下：

mysql> analyze table actor drop histogram on first_name\G
*************************** 1. row ***************************
Table: sakila.actor
Op: histogram
Msg_type: status
Msg_text: Histogram statistics removed for column 'first_name'.
1 row in set ( 0. 01 sec)


### 查看直方图的统计信息

可以在information_schema.column_statistics视图中查看，其中的histogram字段是json格式的文档：

m y s q l> s heilsetcotg srcahme-m>>a'_$n.a"hmiset,o tgarbalme_-tnyapme"e' , (^) acso hluimstno_gnraamm_et,ype,
c a sat(sh disattoegtirmame(- 6 >)>)' $a.s" llaasstt-_uuppddaatetedd"',
c a sat(sh disetcoigmraaml( 4 - >, 2 >)')$ .a"ss asmamplpinlign-gr_artaet"e',
j s o na_s lenunmgtbhe(hr_isotfo_gbruacmk-e>t's$,.buckets')
caass nt(uhmisbteorg_roafm_-b>u'c$k."entus_msbpeerc-oiffi-ebduckets-specified"'as unsigned)
(^) * (^) * (^) * (^) * (^) * (^) *f*r*o*m** (^) *in**fo**r*m*a**t*io**n*_*s*c* h 1 e.m roaw.c o**lu*m**n**_*s*t*a*t*i*s*t*ic**s*\*G********
S TCAHBELMEA_N_NAAMMEE: a: (^) cstaokrila
(^) h (^) i (^) sCtOogLrUaMmN_t_yNpAeM: eEq: (^) ufii-rhset_ignhatme
(^) slaamstp_luinpgd_artaetde:: (^210). 2010 - 08 - 13 09 : 43 : 33. 273680
(^) n (^) u (^) m (^) b (^) e (^) rn_uomf_bbeurc_koeft_sb_uscpkeectisf:ie (^1) d (^0) : 0100
可以看到默认使用了姚 1 远 00 个^ 桶^ 微，信创号建：了ya一oy个ua等na高ce的^ 直^ 方^ 图^ 个。人公众号：数据慧眼


### 把桶的个数增加到 200 时创建直方图

mysql> analyze table actor update histogram on first_name with 200 buckets;
*************************** 1. row ***************************
Table: sakila.actor
Op: histogram
Msg_type: status
Msg_text: Histogram statistics created for column 'first_name'.
1 row in set ( 0. 01 sec)


### 再次检查生成的统计信息

m y s q l> s heilsetcotg srcahme-m>>a'_$n.a"hmiset,o tgarbalme_-tnyapme"e' , (^) acso hluimstno_gnraamm_et,ype,
c a sat(sh disattoegtirmame(- 6 >)>)' $a.s" llaasstt-_uuppddaatetedd"',
cast(histogram->>'$."sampling-rate"'
(^) js (^) o ans_ (^) ldeencgitmh(ahli(s (^4) t,o (^2) g))r (^) aams- s>a'$m.bpulicnkge_trsa't)e,
(^) c (^) a (^) sats(h nisutmogbrearm_o->f_'$b.u"cnkuemtsb,er-of-buckets-specified"'as unsigned)
as number_of_buckets_specified
(^) * (^) * (^) * (^) * (^) * (^) *f*r*o*m** (^) *in**fo**r*m*a**t*io**n*_*s*c* h 1 e.m roaw.c o**lu*m**n**_*s*t*a*t*i*s*t*ic**s*\*G********
S TCAHBELMEA_N_NAAMMEE: a: (^) cstaokrila
COLUMN_NAME: first_name
h ilsatsotg_ruapmd_attyepde: : 2 s 0 i 2 n 1 g-l 0 e 8 to-n 13 09 : 44 : 36. 697382
(^) n (^) u (^) msabmerp_loinf_gb_uractkee: (^) t (^1) s:. 01030
number_of_buckets_specified: 200
发现此时创建的直方姚图远变 成 了 微等信宽号的：，ya分oy配ua的na (^2) c (^0) e 0 个 桶 只 个用人了公 (^13) 众 (^0) 号个：。数据慧眼


### 查询一下这个字段的唯一值的个数

mysql>selectcount(distinctfirst_name)fromactor;
+----------------------------+
|count(distinctfirst_name)|
+----------------------------+
| 130 |
+----------------------------+
1 rowinset( 0. 00 sec)
发现正好也是 130 个，这说明了在等宽的直方图里一个值对应于一个桶。


### 直方图的适用场景

直方图在某些场景下可以帮助优化器生成更优的执行计划，那么在什么样的字段上考虑使用直方图
呢，这里建议符合下面 4 个条件字段可以考虑建立直方图统计信息：
（ 1 ）值分布不均匀，优化器很难估计值的分布的字段。
（ 2 ）选择性差的字段，否则索引更适合。
（ 3 ）用于where子句中过滤的字段或用于连接的字段。
（ 4 ）字段值分布规律不随时间变化的字段。因为直方图统计信息不会自动收集，如果字段值分布
规律发生大的变化，统计信息会失真。
实际工作中，可以使用explain analyze分析SQL语句的执行计划，如果估算的rows和实际的rows
相差过大，可以考虑在过滤字段上创建直方图统计信息。


### 最新课程请联系我


## 姚远

### 多表连接的优化


### 连接优化是 SQL 优化的重要组成部分

表连接的顺序对性能的影响很大，N个表连接时不同的连接顺序的组合是N！（n的阶乘），当N是 5
时，这个组合是 5! = 5 × 4 × 3 × 2 × 1 = 120 。不同的连接顺序的性能相差可能是数量级的。
连接优化的方法：

- 选择小表作为驱动表
- 增大连接缓存


### 选择小表作为驱动表

选择小表作为驱动表可以减少访问被探测表的次数，这里指的小表是指应用了查询限制条件后结果
集小的表。例如下面这个SQL语句的执行计划：

mciytys.qlal>s (^) te_xuppldaainte a=n' 2 a 0 ly 0 z 6 e- 0 s 2 e-l 1 e 5 c t 0 c 4 o: 4 u 5 n:t 2 (* 5 )' f\rGom city inner join country using (country_id) where
*************************** 1. row ***************************
EXPLAIN: - > Aggregate: count( 0 ) (cost= 87. 75 rows= 60 ) (actual time= 2. 146 .. 2. 146 rows= 1 loops= 1 )

- > Nested loop inner join (cost= 81. 75 rows= 60 ) (actual time= 0. 082 .. 2. 027 rows= 600 loops= 1 )

(^) t (^) im (^) e (^) = 0 - >. 0 F 6 i 7 lt.e. 0 r:. 8 ( 8 c 1 it yr.olawsst=_ 6 u 0 p 0 d aloteo (^) p=s T=I 1 M)ESTAMP' 2006 - 02 - 15 04 : 45 : 25 ') (cost= 60. 75 rows= 60 ) (actual

- > Table scan on city (cost= 60. 75 rows= 600 ) (actual time= 0. 061 .. 0. 349 rows= 600 loops= 1 )

(^) ( (^) a (^) c (^) t (^) u (^) a-l> t (^) iSmineg=l 0 e.- 0 r 0 o 1 w. (^) .i 0 n.d 0 e 0 x 1 lrooowksu=p 1 o lno (^) ocposu=n 6 tr 0 y 0 u)sing PRIMARY (country_id=city.country_id) (cost= 0. 25 rows= 1 )
1 row in set ( 0. 00 sec)


### 选择小表作为驱动表

查询city表中的全部记录个数和符合过滤条件的记录个数如下：
mysql> select count(*),sum(last_update=' 2006 - 02 - 15 04 : 45 : 25 ') from city;
+----------+----------------------------------------+
| count(*) | sum(last_update=' 2006 - 02 - 15 04 : 45 : 25 ') |
+----------+----------------------------------------+
| 600 | 600 |
+----------+----------------------------------------+
1 row in set ( 0. 00 sec)
查询另一个连接表country中的记录个数如下：
mysql> select count(*) from country;
+----------+
| count(*) |
+----------+
| 109 |
+----------+
1 row in set ( 0. 01 se姚c远) 微信号：yaoyuanace 个人公众号：数据慧眼


### 与连接顺序相关的提示

- JOIN_FIXED_ORDER：按from子句中表的排序顺序进行表连接，这个提示和SELECT
    STRAIGHT_JOIN的功能一样。
- JOIN_ORDER：按提示指定的表顺序连接表。
- JOIN_PREFIX：把提示中指定的表作为连接时的第一个表，并按提示里的顺序进行连接。
- JOIN_SUFFIX：把提示中指定的表作为连接时的最后一个表，并按提示里的顺序进行连接。


### 通过 JOIN_ORDER 提示设置表的连接顺序

m(cyosuqnlt>r (^) ye_xidpl)a winh (^) earnea lcyiztye. (^) lsaeslte_cutp /d*a+t (^) eJ=O'I 2 N 0 _ 0 O 6 - R 0 D 2 E- 1 R 5 ( c 0 o 4 u: 4 n 5 tr:y 2 , 5 c'i t\yG) */ count(*) from city inner join country using
*************************** 1. row ***************************
EXPLAIN: - > Aggregate: count( 0 ) (cost= 227. 15 rows= 60 ) (actual time= 1. 911 .. 1. 911 rows= 1 loops= 1 )

- > Nested loop inner join (cost= 221. 15 rows= 60 ) (actual time= 0. 066 .. 1. 825 rows= 600 loops= 1 )

(^) l (^) o (^) o (^) p (^) s (^) =-> 1 )Index scan on country using PRIMARY (cost= 11. 15 rows= 109 ) (actual time= 0. 038 .. 0. 068 rows= 109
(^) t (^) im (^) e (^) = 0 - >. 0 F 0 i 7 lt.e. 0 r:. 0 ( 1 c 5 it yr.olawsst=_ 6 u plodoaptes (^) == 1 T 0 I 9 M)ESTAMP' 2006 - 02 - 15 04 : 45 : 25 ') (cost= 1. 38 rows= 1 ) (actual
(^) ( (^) a (^) c (^) t (^) u (^) a (^) l (^) t (^) i-m> (^) eI=n 0 d.e 0 x 0 l 5 o.o. 0 k.u 0 p 1 1 o nr (^) ocwitsy= u 6 s lionogp isd=x 1 _ 0 fk 9 _)country_id (country_id=country.country_id) (cost= 1. 38 rows= 6 )
1 row in set ( 0. 00 sec)


### 比较这两种连接方式的性能

mysql> select left(sql_text, 40 ), rows_examined,timer_wait/ 1000000000 ms from
performance_schema.events_statements_history where sql_text like '%city inner join
country%' and thread_id!=ps_current_thread_id() order by timer_start desc limit 2 ;
+------------------------------------------+---------------+--------+
|left(sql_text, 40 )|rows_examined|ms|
+------------------------------------------+---------------+--------+
|selectcount(*)fromcityinnerjoincou| 1200 | 2. 7704 |
|select/*+JOIN_ORDER(country,city)*/c| 709 | 2. 2053 |
+------------------------------------------+---------------+--------+
2 rows in set ( 0. 01 sec)


### 比较执行性能

发现估算成本高的SQL执行时间反而短：


### 增大连接缓存

连接缓存是MySQL用来在连接时进行数据缓存的区域。每次连接使用一个连接缓存，因此执行一个
SQL语句可能用到多个连接缓存，连接缓存在SQL语句执行之前分配，执行完成后释放。每个连接
缓存的大小由系统参数join_buffer_size决定，对于这个参数的设置，长期以来的建议是：初始值
可以分配得小一些，对于需要大的连接缓存的会话和SQL语句可以单独进行调整，如果统一设置得
很大，对于很多SQL语句实际上是浪费。但在MySQL 8. 0. 18 以后的版本，连接缓存的分配是根据
需要进行递增分配，join_buffer_size只是连接缓存的上限，但外连接要分配全部的连接缓存，从
MySQL 8. 0. 20 以后，包括外连接对连接缓存的需求也可以进行递增的分配了，因此设置一个较大
的join_buffer_size已经不会有什么副作用了。


### 设置连接缓存

系统参数join_buffer_size默认值是 256 KB，下面分别是在会话级和全局级把它设置为 1 GB的例子：

mysql> set join_buffer_size= 1024 * 1024 * 1024 ;
mysql> set global join_buffer_size= 1024 * 1024 * 1024 ;
也可以使用set_var提示对单个SQL语句调节join_buffer_size的大小。


### 使用默认连接缓存时执行 SQL

当前会话的连接缓存默认是 256 KB，执行一个连接SQL语句如下：

mysql> select count(*) from sbtest 1 inner join sbtest 2 using (c);
+----------+
|count(*)|
+----------+
| 0 |
+----------+
1 row in set ( 7. 57 sec)


### 使用默认连接缓存时执行 SQL

m**y**s*q*l*>* (^) *e*x**p*l*a*i*n* (^) * s**e*l*e*c*t* (^) *c*o 1 u.n tr(o*w) f*r*o*m** (^) *s*b*t*e*s**t (^1) ** i*n*n**e*r* (^) *jo**in** (^) *s*btest 2 using (c)\G
(^) s (^) e (^) l (^) e (^) c (^) t (^) _idty: (^) p (^1) e: SIMPLE
(^) p (^) a (^) r (^) ttiatibolnes: (^) :s NbtUeLsLt 1
(^) p (^) o (^) s (^) s (^) i (^) b tlyep_ek:e (^) yAsL:L NULL
(^) k (^) e (^) yk_elye:n N: NULULLL
(^) rroewf:s (^) :N 8 U 3 L 3 L 33
f i l tEexretrda:: (^1) N (^0) U (^0) L. (^0) L 0
* * * * * * * * *i*d*:* * 1 ************** 2. row ***************************
s e l e ctat_btlyep: es:b SteIsMtP 2 LE
p a r t ittyiopnes: :A NLULLL
p o s s i b l eke_yk:e yNsU: (^) LNLULL
k e yr_elfe: nN: (^) UNLULLL
(^) f (^) il (^) t (^) erorewds:: 1605. 705076
(^2) r (^) o (^) w (^) sE xintr as:e (^) tU, s 1 i nwga wrnhienrge ;( 0 U. 0 si 0 n gse jco)in buffer (hash join)
发现它使用了哈希连接姚，远驱 动 表 是微s信bt号es：t (^1) y，aoyuanace 个人公众号：数据慧眼


### 使用默认连接缓存时执行 SQL

查my看sq驱l>动 s表h的ow状 t态ab如le下 s：tatus like 'sbtest 1 '\G
* * * * * * * * *N**a*m**e*:* *s*b**te**s*t* 1 *** 1. row ***************************

(^) VEenrgsiinoen:: M 10 yISAM
R o w _Rfoowrms:a t 8 : 3 F 3 i 3 x 3 ed
A v Dga_rtao_wl_elnegntght:h 6 : (^0772499757)
M aInx_ddeax_tale_nlegntght:h 1 : 720025911925258022068223
(^) A (^) u (^) t (^) oD_aitnac_rfermeee:n (^0) t: 1000001
CUrpedaattee__ttiimmee:: 22002211 - - 0055 - - 3311 1290 :: 1202 :: (^3353)
C Choelclakt_iotinm:e u: (^) t (^2) f 80 m (^2) b 14 - (^0) _ 50 - 93010 2 _ (^0) a:i (^2) _ (^2) ci: 33
(^) C (^) r (^) e (^) aCteh_eocpktsiuomns: (^) :N ULL
(^1) r (^) o (^) w Cino msemt e(n 0 t.: 0 1 sec)
这个驱动表的大小大约是 (^6) 姚 (^0) 远M， 把 根 据微这信个号表：的cy字ao段yu生a成na的c哈e 希 表 放 入 2 个 (^56) 人K的公连众接号缓：存中数显据然慧放眼不下，


### 增大连接缓存后再执行这个 SQL

在SQL语句里通过提示设置join_buffer_size的大小为 60 M，进行对比测试如下：

mysql> select /*+ set_var(join_buffer_size= 60 M)) */ count(*) from sbtest 1 inner join
sbtest 2 using (c);
+----------+
|count(*)|
+----------+
| 0 |
+----------+
1 rowinset( 1. 57 sec)
可以看到增大join_buffer_size后，执行时间大约缩短了四分之一。


## 姚远

### 如何让排序速度成倍提


### 进行排序的两种实现方式

MySQL中对记录进行排序有两种实现方式：
一种是使用索引进行排序，在执行计划的Extra字段中会有Using index的信息。
另一种是不使用索引进行排序，称之为文件排序（filesort），在执行计划的Extra字段中会有
Using filesort的信息。在多表连接的时候，如果需要保存中间排序结果进行连接，Extra字段中会
有“Using temporary; Using filesort“ 的信息。


### 进行排序的两种实现方式

例如actor表的last_name字段上有索引，而first_name字段上没有索引，对这两个字段分别进行排序，两个执行计划如下：
mysql> explain select last_name from actor order by last_name\G
...
Extra: Using index
1 row in set, 1 warning ( 0. 00 sec)
mysql> explain select first_name from actor order by first_name\G
...
Extra: Using filesort
1 row in set, 1 warning ( 0. 00 sec)
可以看到对有索引的字段进行排序，使用了索引排序；对没有索引的字段进行排序，使用了文件排序。


### 系统参数 sort_buffer_size

文件排序使用的内存大小由系统参数sort_buffer_size决定，默认是 256 K，这对大数据量的排序是
不够用的。在早期的版本里，sort_buffer_size设定的是固定的排序缓存大小。从MySQL 8. 0. 12
开始，sort_buffer_size设定的是排序缓存的上限，对于排序过程中使用的内存是根据需要递增分
配的，因此把它设置成一个较大的值并没有副作用。排序时需要的排序缓存可能比预计的大得多，
因为MySQL会给每条记录都会分配一个能容纳最大记录的内存，例如varchar类型的字符串是按照
它的完整长度分配空间，对于变长的UTF字符集也是按最长的字节分配空间。如果排序缓存不够，
MySQL会将数据分块排序，然后进行合并，这个过程中会在磁盘上生成临时文件，因此效率会大大
下降，可以从状态参数Sort_merge_passes中看到合并的次数，这个值最好是 0 ，如果过大，建议
增加sort_buffer_size的设置。


### 默认系统参数 sort_buffer_size

首先重置状态变量：
mysql> flush status;
Query OK, 0 rows affected ( 0. 01 sec)
执行一个排序的SQL语句如下：
mysql> select * from sbtest 2 order by c;
...
666666 rows in set ( 8. 72 sec)

```
查询与排序相关的状态变量如下：
mysql>SHOWSTATUSLIKE
'sort%';
+-------------------+--------+
|Variable_name|Value|
+-------------------+--------+
|Sort_merge_passes| 193 |
|Sort_range| 0 |
|Sort_rows| 666666 |
|Sort_scan| 1 |
+-------------------+--------+
4 rowsinset( 0. 00 sec)
```

### 减少查询的字段

```
第二次查询与排序相关的状态变量如下：
mysql>SHOWSTATUSLIKE'sort%';
+-------------------+--------+
|Variable_name|Value|
+-------------------+--------+
|Sort_merge_passes| 165 |
|Sort_range| 0 |
|Sort_rows| 666666 |
|Sort_scan| 1 |
+-------------------+--------+
4 rowsinset( 0. 01 sec)
```
第二次重置状态变量：

mysql> flush status;

Query OK, 0 rows affected ( 0. 01 sec)

第二次执行这个排序的SQL语句如下：

mysql> select c from sbtest 2 order by
c;

...

666666 rows in set ( 7. 59 sec)


### 增大排序缓存

mysql> select /*+ set_var(sort_buffer_size= 1 G) */ c from sbtest 2 order by c;
...
666666 rows in set ( 2. 77 sec)
使用 1 G的排序缓存执行这个SQL语句后的排序状态变量如下：
mysql>SHOWSTATUSLIKE'sort%';
+-------------------+--------+
|Variable_name|Value|
+-------------------+--------+
|Sort_merge_passes| 0 |
|Sort_range| 0 |
|Sort_rows| 666666 |
|Sort_scan| 1 |
+-------------------+--------+
4 rows in set ( 0. 01 s姚e远c)^ 微信号：yaoyuanace^ 个人公众号：数据慧眼


### 比较 3 个 SQL 语句的执行性能

mysql>selectleft(SQL_TEXT, 40 ),sort_merge_passes,TIMER_WAIT/ 1000000000 MSfrom
performance_schema.events_statements_historywhereSQL_TEXTlike'%orderby%'and
THREAD_ID!=PS_CURRENT_THREAD_ID()orderbyTIMER_STARTdesclimit 3 ;
+------------------------------------------+-------------------+-----------+
|left(SQL_TEXT, 40 )|sort_merge_passes|MS|
+------------------------------------------+-------------------+-----------+
|select/*+set_var(sort_buffer_size= 1 G| 0 | 2768. 8907 |
|selectcfromsbtest 2 orderbyc| 165 | 7590. 2911 |
|select*fromsbtest 2 orderbyc| 193 | 8716. 0335 |
+------------------------------------------+-------------------+-----------+
3 rowsinset( 0. 00 sec)


### 优化索引排序

对需要排序的字段增加索引通常可以让优化器使用索引排序，但不是绝对的，因为优化器要考虑总体的性能，例如下面SQL
语句：
mysql> explain select * from sbtest 2 order by k\G
...
Extra: Using index
1 row in set, 1 warning ( 0. 00 sec)
虽然k字段上有索引，优化器仍然采用了filesort，因为这个SQL语句查询了表里的所有字段，如果只查询k字段：
mysql> explain select k from sbtest 2 order by k\G
...
Extra: Using filesort
1 row in set, 1 warning ( 0. 00 sec)
这个时候优化器就采用了索引排序。因此在写SQL语句时，不要简单地使用一个星号查询所有的字段，应该把需要的字段在
select后面一个一个地列出来，即使的确需要查询所有的字段，严谨的做法也是把字段一个一个地都列出来，因为将来表结
构的变化，可能会增加姚、远修^ 改^ 或^ 删微除信字号段：。yaoyuanace^ 个人公众号：数据慧眼


## 姚远

### 表空间碎片整理


### 表空间碎片整理的内容

##### 1. 检查表空间碎片

##### 2. 整理表空间与性能提升

##### 3. 使用mysqlcheck进行批量表空间优化


### 表空间碎片的产生

MySQL的表在进行了多次delete、update和insert后，表空间会出现碎片。定期进行表空间整理，
消除碎片可以提高访问表空间的性能。


### 没有碎片的表

收集一个有 100 万记录表的统计信息的命令和输出结果如下：
mysql>analyzetablesbtest 1 ;
+----------------+---------+----------+-----------------------------+
|Table|Op|Msg_type|Msg_text|
+----------------+---------+----------+-----------------------------+
|sbtest.sbtest 1 |analyze|status|Tableisalreadyuptodate|
+----------------+---------+----------+-----------------------------+
1 rowinset( 0. 06 sec)
查询这个表的状态信息如下：
mysql>showtablestatuslike'sbtest 1 '\G
*************************** 1 .row***************************
...
Max_data_length: 205195258022068223
Index_length: 20457472
Data_free: 0
...

```
姚远 微信号：yaoyuanace 个人公众号：数据慧眼
```

### 从操作系统层查询对应数据文件

再从操作系统层查询对应数据文件的大小如下：

mysql> system ls - l /var/lib/mysql/sbtest/sbtest 1 .*

- rw-r----- 1 mysql mysql 729000000 May 31 08 : 24 /var/lib/mysql/sbtest/sbtest 1 .MYD
- rw-r----- 1 mysql mysql 20457472 May 31 08 : 25 /var/lib/mysql/sbtest/sbtest 1 .MYI
命令show table status和从操作系统层看到的数据文件大小一致，这时的Data_free为零。


### 删除这个表三分之二的记录

下面的命令删除这个表三分之二的记录：
mysql>deletefromsbtest 1 whereid% 3 <> 0 ;
QueryOK, 666667 rowsaffected( 51. 72 sec)
重新收集这个表的统计信息的命令和输出结果如下：
mysql>analyzetablesbtest 1 ;
+----------------+---------+----------+----------+
|Table|Op|Msg_type|Msg_text|
+----------------+---------+----------+----------+
|sbtest.sbtest 1 |analyze|status|OK|
+----------------+---------+----------+----------+
1 rowinset( 0. 13 sec)


### 再次检查表的状态

mysql>showtablestatuslike'sbtest 1 '\G
*************************** 1 .row***************************
...
Data_free: 486000243
...
可以看到这次Data_free不再是 0 了，计算Data_free和Data_length的比率如下：
mysql>select 486000243 / 729000000 ;
+---------------------+
| 486000243 / 729000000 |
+---------------------+
| 0. 6667 |
+---------------------+
1 rowinset( 0. 00 sec)
发现空闲空间占用了总姚空远间 的 三 分微之信二号。：yaoyuanace 个人公众号：数据慧眼


### 再次从操作系统层查询对应数据文件

再次从操作系统层查询对应数据文件的大小如下：

mysql> system ls - l /var/lib/mysql/sbtest/sbtest 1 .*

- rw-r----- 1 mysql mysql 729000000 May 31 08 : 33 /var/lib/mysql/sbtest/sbtest 1 .MYD
- rw-r----- 1 mysql mysql 20457472 May 31 08 : 34 /var/lib/mysql/sbtest/sbtest 1 .MYI
发现虽然这个表中的三分之二的记录已经被删除，但数据文件的大小还和原来一样。


### 整理表空间

下面的SQL语句进行表空间整理：

mysql> alter table sbtest 1 force;
Query OK, 333333 rows affected ( 10. 73 sec)
Records: 333333 Duplicates: 0 Warnings: 0
整理完成后，再次收集这个表的统计信息的命令和输出结果如下：
mysql> analyze table sbtest 1 ;
第 3 次查看表的状态信息如下：
mysql> show table status like 'sbtest 1 '\G
...
Data_free: 0


### 第 3 次从操作系统层查询对应数据文件

第 3 次从操作系统层查询对应数据文件的大小如下：

mysql> system ls - l /var/lib/mysql/sbtest/sbtest 1 .*

- rw-r----- 1 mysql mysql 242999757 May 31 08 : 40 /var/lib/mysql/sbtest/sbtest 1 .MYD
- rw-r----- 1 mysql mysql 6820864 May 31 08 : 40 /var/lib/mysql/sbtest/sbtest 1 .MYI
发现经过整理后，硬盘空间占用只剩下原来的三分之一，被删除的记录的硬盘空间都释放了。


### 再次执行全表扫描

mysql> select count(*) from sbtest 1 where c<>'aaa';
+----------+
| count(*) |
+----------+
| 333333 |
+----------+
1 row in set ( 0. 29 sec)
发现执行速度也提高到大约原来的三倍。


### 补充说明

这里使用的是MyISAM表进行测试，如果用InnoDB表，速度的提高没有这么明显，因为InnoDB的
数据会缓存到InnoDB缓存中，MyISAM表的数据MySQL不进行缓存，OS可能会缓存，因此要得到
准确的测试结果，在Linux系统上每次测试前要使用下面的命令释放系统的缓存：

# echo 3 > /proc/sys/vm/drop_caches


### 补充说明

使用alter table force进行表空间整理和optimize table命令的作用一样，这个命令适用于 InnoDB,
MyISAM和ARCHIVE三种引擎的表。但对于InnoDB的表，不支持optimize table命令，例如：

mysql> optimize table sbtest 2 \G
...
Msg_text: Table does not support optimize, doing recreate + analyze instead
可以用alter table sbtest 1 engine=innodb代替，例如：
mysql> alter table sbtest 2 engine=innodb;


### 检查表空间的碎片

下面是找出表空间中可释放空间超过 2 M的最大 10 个表的SQL命令和输出结果：

mysql> select TABLE_SCHEMA,table_name,round(data_length/ 1024 / 1024 ) as
data_length_mb, round(data_free/ 1024 / 1024 ) as data_free_mb from
information_schema.tables where round(data_free/ 1024 / 1024 ) > 2 and table_schema not
in ('mysql') order by data_free_mb desc limit 10 ;
+------------+----------------+--------------+
|TABLE_NAME|data_length_mb|data_free_mb|
+------------+----------------+--------------+
|sbtest 2 | 232 | 174 |
+------------+----------------+--------------+
1 rowinset( 0. 02 sec)


### mysqlcheck 工具

可以使用MySQL自带的工具mysqlcheck的-o选项进行表空间优化，这个工具适合于在脚本中进行
批量处理，可以被Linux中的crontab或Windows中的计划任务调用。
先使用-a进行统计信息收集，再使用-o进行表空间优化。
对单个表进行表空间优化的命令如下：
$ mysqlcheck - o sbtest sbtest 1
也可以使用下面的命令对某个数据库中的所有表进行表空间优化：
$ mysqlcheck - o sbtest
对整个实例中对所有数据库进行表空间优化的命令如下：
$ mysqlcheck - o - -all-databases
注意这种方式不支持InnoDB的表，可以用alter table tb engine=innodb代替。


## 姚远

### 让 SQL 优雅且高效的 CTE

##### MySQL 8 性能优化


### CTE 简介

MySQL从 8. 0 开始支持CTE（Common Table Expression），CTE是一个命名的临时结果集，它
存在于单个语句的范围内，并且可以在该语句中多次引用，而且CTE还可以相互引用。在MySQL 8
之前，进行复杂查询时需要使用子查询来实现，造成SQL语句复杂、性能低，而且可读性差。CTE
的出现简化了复杂查询语句的编写，提高了SQL性能。
CTE的基本语法如下：
WITH cte_name (column_list) AS (
query
)
SELECT * FROM cte_name;
查询（query）中的字段数必须与column_list中的字段数相同。如果省略column_list，CTE将使
用查询中的列。


### CTE 的例子 1

例如下面的SQL语句查询所有的中国城市：

select city from city where country_id=(select country_id from country where
country=‘China’) ;
改写成CTE：
with country_cn as (
select country_id from country where country='China')
select city from city inner join country_cn using (country_id);


### CTE 的例子 2

查询出租电影每个月的收入的SQL语句和输出结果：
mysql> select date_format(r.rental_date, '%y-%m- 01 ') as month 01 , sum(p.amount) as
revenue from sakila.payment p inner join sakila.rental r using (rental_id) group by
month 01 ;
+----------+----------+
| month 01 | revenue |
+----------+----------+
| 05 - 05 - 01 | 4823. 44 |
| 05 - 06 - 01 | 9629. 89 |
| 05 - 07 - 01 | 28368. 91 |
| 05 - 08 - 01 | 24070. 14 |
| 06 - 02 - 01 | 514. 18 |
+----------+----------+
5 rows in set ( 0 .姚 09 远 s^ e^ c^ )^ 微信号：yaoyuanace^ 个人公众号：数据慧眼


### CTE 例子 2

一个常见的需求是对比每个月的收入相对于上个月的收入增长情况，可以把查询每个月的收入的
SQL块写成一个CTE，两次引用这个CTE，一次用作当前月，另一次用作上个月，对这两次引用进
行左外连接，计算出相邻两个月收入对差，相应的SQL语句和输出结果如下：
mysql> WITH monthly_sales(FirstOfMonth, Sales) AS (SELECT
DATE_FORMAT(r.rental_date, '%Y-%m- 01 ') AS FirstOfMonth, SUM(p.amount) as Sales
FROM sakila.payment p INNER JOIN sakila.rental r USING (rental_id) GROUP BY
FirstOfMonth) SELECT YEAR(cur.FirstOfMonth) AS 'Year',
MONTHNAME(cur.FirstOfMonth) AS 'Month', cur.Sales, (cur.Sales - IFNULL(prev.Sales,
0 )) AS Delta FROM monthly_sales cur LEFT OUTER JOIN monthly_sales prev ON
prev.FirstOfMonth = cur.FirstOfMonth - INTERVAL 1 MONTH ORDER BY cur.FirstOfMonth;


### 不使用 CTE 时的 SQL 语句

mysql> SELECT YEAR(cur.FirstOfMonth) AS 'Year', MONTHNAME(cur.FirstOfMonth) AS
'Month', cur.Sales, (cur.Sales - IFNULL(prev.Sales, 0 )) AS Delta FROM (SELECT
DATE_FORMAT(r.rental_date, '%Y-%m- 01 ') AS FirstOfMonth, SUM(p.amount) as Sales
FROM sakila.payment p INNER JOIN sakila.rental r USING (rental_id) GROUP BY
FirstOfMonth) cur LEFT OUTER JOIN ( SELECT DATE_FORMAT(r.rental_date, '%Y-%m-
01 ') AS FirstOfMonth, SUM(p.amount) as Sales FROM sakila.payment p INNER JOIN
sakila.rental r USING (rental_id) GROUP BY FirstOfMonth) prev ON prev.FirstOfMonth =
cur.FirstOfMonth - INTERVAL 1 MONTH ORDER BY cur.FirstOfMonth;


### 使用 CTE 的执行计划
```
| - > Nested loop left join (cost= 40325. 00 rows= 0 )

- > Sort: cur.FirstOfMonth
- > Table scan on cur (cost= 1816. 56 rows= 16125 )
- > Materialize CTE monthly_sales if needed (cost= 0. 00 .. 0. 00 rows= 0 )
- > Table scan on <temporary>
- > Aggregate using temporary table
- > Nested loop inner join (cost= 7280. 50 rows= 16125 )
- > Filter: (p.rental_id is not null) (cost= 1636. 75 rows= 16125 )
- > Table scan on p (cost= 1636. 75 rows= 16125 )
    - > Single-row index lookup on r using PRIMARY (rental_id=p.rental_id) (cost= 0. 25 rows= 1 )
- > Filter: (`prev`.FirstOfMonth = (cur.FirstOfMonth - interval 1 month)) (cost= 0. 25 .. 2. 50 rows= 10 )
- > Index lookup on prev using <auto_key 0 > (FirstOfMonth=(cur.FirstOfMonth - interval 1 month))
- > Materialize CTE monthly_sales if needed (query plan printed elsewhere) (cost= 0. 00 .. 0. 00 rows= 0 )
|
```

### 不使用 CTE 的执行计划
```
- > Nested loop left join (cost= 40325. 00 rows= 0 )
- > Sort: cur.FirstOfMonth
- > Table scan on cur (cost= 1816. 56 rows= 16125 )
- > Materialize (cost= 0. 00 .. 0. 00 rows= 0 )
- > Table scan on <temporary>
- > Aggregate using temporary table
- > Nested loop inner join (cost= 7280. 50 rows= 16125 )
- > Filter: (p.rental_id is not null) (cost= 1636. 75 rows= 16125 )
- > Table scan on p (cost= 1636. 75 rows= 16125 )
- > Single-row index lookup on r using PRIMARY (rental_id=p.rental_id) (cost= 0. 25 rows= 1 )
- > Filter: (`prev`.FirstOfMonth = (cur.FirstOfMonth - interval 1 month)) (cost= 0. 25 .. 2. 50 rows= 10 )
- > Index lookup on prev using <auto_key 0 > (FirstOfMonth=(cur.FirstOfMonth - interval 1 month))
- > Materialize (cost= 0. 00 .. 0. 00 rows= 0 )
- > Table scan on <temporary>
- > Aggregate using temporary table
- > Nested loop inner join (cost= 7280. 50 rows= 16125 )
- > Filter: (p.rental_id is not null) (cost= 1636. 75 rows= 16125 )
- > Table scan on p (cost= 1636. 75 rows= 16125 )
- > Si姚ngl远e-r^ o^ w^ i^ n^ d^ 微ex 信loo号ku：p oyna ro uysuinagn PaRcIeM^ A^ R^ Y^ (^ r^ e^ n^ t^ a个l_人id=公p.r众en号tal：_id数) 据(co慧st=眼 0. 25 rows= 1
```

### 对比两个 SQL 语句的执行计划

对比使用CTE改写后的SQL语句的执行计划和传统的SQL语句的执行计划：
传统的SQL语句把CTE里面的SQL块两次物化（Materialize）成了临时表，而使用CTE改写后的
SQL语句物化CTE时使用的是 “Materialize CTE monthly_sales if needed”，实际上只物化了
一遍。
从这个例子里可以看到，对于复杂的SQL语句使用CTE改写后可读性和执行效率都大大提高。


### 最新课程请联系我


