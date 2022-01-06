# MySQL 参数优化

全文中一共有常用的（事实上你如果花1-2周阅读、理解、自己动手设一下后是需要这么多参数的）**76**个参数，笔者把近10年里3个亿万级项目的数据库调优用此篇浓缩到了可能读者只需要2周时间就可以掌握，同时我是按照：

每一个参数干吗？
在某些典型硬件配置下的db上参数该设多少？
设会怎么样？
不设会怎么样？
有什么坑如何填坑？
有些参数怎么算、算法又如何
这种style来写的，相信此篇会对一些使用mysql的尤其是正在或者将要面临万级并发的项目、网站有所帮助。具体请看文档！

##### 一千个DBA就有一千种配置方式!

大家一定记得不要轻易去看网上，要看只看官网！网上很多博客都是错的，连参数都列错了，5.7很多参数和5.6是完全不一样的。

可能你从未看到过这样的一篇集中火力式的把mysql参数列了这么全的文章，很有兴曾参与过超3万并发的18～19年的数轮520、618、双11、双12保卫战。因此这一篇是汇集了最精华和实战的内容把mysql所有的参数列在这边供大家参考。并且以（64c cpu，128gb内存）的mysql cpu和内存来进行了一轮配置。而此文的内存相关参数部分可以延展至256gb～512gb。

另外有一点，建议在mysql的服务器上使用ssd。除非并发数永远控制在500-1000内那就没必要使用ssd，普通高速磁盘就可以了。

你会发觉这篇文章是一篇宝藏，这些参数都能够自己动手试验一篇基本在外面是可以吊打mysql面试官了。

### client域：
##### 1.character_set_client
推荐设置：

utf8mb4

作用：

字符集设定，如果前台有连social mobile application一类包括wechat，并且允许有使用emoji表情的，请开启成utf8mb4

如果不配的后果：

mysql不支持前端app存表情等字符

配置实例：

character_set_client=utf8mb4

### mysqld域：
##### 4.autocommit
推荐设置：

作用：

生产上开启成1，如果你开启的是0会有一个这样的情况：

a运行一条insert语句，并未作commit;b去做查询此时b是查询不到的。这种操作一般用于在写store procedure时用到。

如果不配的后果：

如果在系统的my.cnf层面把它设成了0，如果在使用时（99%情况是用的1）时，你想要用root在生产运行时把它设成set autocommit = 1都开启不了。而如果你在一开始就没它设置成1，那么当碰到某些特殊场景特别是写store procedure时需要把它设成0时，你是可以手动临时把某一个session给开在0的。

配置实例：

autocommit = 1

##### 5.character_set_server
推荐设置：

utf8mb4

作用：

字符集设定，如果前台有连social mobile application一类包括wechat，并且允许有使用emoji表情的，请开启成utf8mb4

如果不配的后果：

mysql不支持前端app存表情等字符

配置实例：

character_set_server=utf8mb4

##### 6.skip_name_resolve
推荐设置：

1

作用：

生产上建议开启成1，这样mysql server不会对客户端连接使用反向dns解析，否则客户端连上后有时在遇有生产高速运行时直接timeout，如果设成了1带来的问题就是你不能在mysql中使用主机名来对客户端权限进行划分，而是需要使用ip。

如果要做成即允许mysql里允许使用主机名来分配客户端连接权限，又要做到不要让mysql去做dns解析，可以在mysql所在主机端的/etc/hosts文件中写上客户端的主机名，因为当客户端连接连上来时，mysql反向查找客户端连接时的域名解析的步骤是：首先查找 /etc/hosts 文件，搜索域名和IP的对应关系。但是这样做也有一个问题，那就是如果你有多个客户端多个mysql主从关系，哪到你要把mysql做成一个dns解析器吗？因此推荐设成1

如果不配的后果：

mysql server每一次会对客户端连接使用反向dns解析，经常会出现客户端连上后有timeout现象。

配置实例：

skip_name_resolve=1

##### 7.max_connections
推荐设置：

5,000

作用：

最大连接数，以微品会：前端3万的tps并发，假设redis命中失效50%（这是灾难），那么后端mysql单个主或从开启连接数为：20,000，我们公司在前端并发曾达到过6万，80%被waf、vanish、缓存挡掉，落在db上的qps最高一次为20,000连接，再按照mysql官方，max_connections值受系统os最大打开连接数限制，因此我们需要做以下2步操作：

1）在 /etc/security/limits.conf 底部增加2行

```
mysql hard nofile 65535
mysql soft nofile 65535
```

2）在/usr/lib/systemd/system/mysqld.service（视如何安装mysql所决定，用编译安装和yum安装会产生path路径不同。）文件最后添加：

```
LimitNOFILE=65535
LimitNPROC=65535
```

```
systemctl daemon-reload
systemctl restart mysqld.service
```

如不生效重服务器。

如果不配的后果：

默认只有150

配置实例：

max_connections = 5,000

##### 9.innodb_flush_log_at_trx_commit
推荐设置：

2

作用：

(核心交易系统设置为1，默认为1，其他2或者0)，

0代表：log buffer将每秒一次地写入log file中，并且log file的flush(刷到磁盘)操作同时进行。该模式下在事务提交的时候，不会主动触发写入磁盘的操作。

1代表：每次事务提交时MySQL都会把log buffer的数据写入log file，并且flush(刷到磁盘)中去，该模式为系统默认（因此会保留每一份redo日志）

2代表：每次事务提交时MySQL都会把log buffer的数据写入log file，但是flush(刷到磁盘)操作并不会同时进行。该模式下，MySQL会每秒执行一次 flush(刷到磁盘)操作。该模式速度较快，也比0安全，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失。
除非你用的是小型机或者是超大规模mysql集群一类如：游戏行业，那么需要保留每一秒的事务，否则请设成2，要不然会严重影响系统性能。这个参数是5.6所没有的。

如果不配的后果：

默认为1，影响系统写性能。

配置实例：

innodb_flush_log_at_trx_commit=2

##### 10.transaction_isolation
推荐设置：

READ-COMMITTED
作用：

此参数直接决定了mysql的性能，oracle中的事务默认级别就是read-commited，而mysql的默认级别是:repeatable-read，它利用自身独有的Gap Lock解决了"幻读"。但也因为Gap Lock的缘故，相比于READ-COMMITTED级别的Record Lock，REPEATABLE-READ的事务并发插入性能受到很大的限制。离级别的选择取决于实际的业务需求（安全与性能的权衡），如果不是金融、电信等事务级别要求很高的业务，完全可以设置成transaction_isolation=READ-COMMITTED。

Repeatable-read: 这是MySQL的InnoDB引擎默认的隔离级别，它阻止查询的任何行被其他事务更改。因此，阻塞不可重复读，而不是幻读。也就是说在可重复读中，可能会出现幻读。重复读使用一种中等严格的锁定策略，以便事务中的所有查询都能看到来自相同快照(即事务启动时的数据)的数据。当拥有该级别的事务执行 UPDATE ... WHERE, DELETE ... WHERE, SELECT ... FOR UPDATE和LOCK IN SHARE MODE操作时，其他事务可能需要等待。
Read-Committed-推荐: 事务无法看到来自其他事务的未提交数据，但可以看到当前事务启动后另一个事务提交的数据。当拥有这种级别的事务执行 UPDATE ... WHERE or DELETE ... WHERE操作时，其他事务可能需要等待。但是该事务可以执行 SELECT ... FOR UPDATE, and LOCK IN SHARE MODE操作，其他事务不需要等待。
串行化（SERIALIZABLE）-极力不推荐，串行化隔离级别是最高的隔离级别，它使用了最保守的锁策略。
读未提交（READ-UNCOMMITTED）-它是最低的隔离级别，虽然性能最高，但也不推荐
它会读取到其他事务修改尚未提交的数据，使用此隔离级别就需要非常小心，认识到这种级别下的查询结果可能不一致或不可复制，这取决于其他事务同时在做什么。通常，具有此隔离级别的事务只执行查询，而不执行插入、更新或删除操作。

在实际环境中，应当根据是否允许出现脏读（dirty reads），不可重复读（non-repeatable reads）和幻读（phantom reads ）现象而选择相应的隔离级别。例如在大数据中，少量的数据不一致不会影响到最后的决策，这种情况下可以使用较低的隔离级别以提交性能和并发性。

如果不配的后果：

默认就是repeatable-read

配置实例：

transaction_isolation = READ-COMMITTED

##### 11.explicit_defaults_for_timestamp
推荐设置：

1
作用：

mysql5.7默认对于timestamp字段会显示“系统当前日期”，就算你在插表时这个timestamp字段留空，它在select出来时也会显示系统日期。因此，这个值的影响范围是你在建表时导致的。

系统默认这个值是0，在0的情况下，你要让该表的timestamp字段在为null时不显示系统默认时间，你的建表必须为：create table order(o_id int ,updateed_time timestamp null default null) ;

explicit_defaults_for_timestamp 变量会直接影响表结构，也就是说explicit_defaults_for_timestamp的作用时间是在表定义的时候；你的update | insert 想通过它去改变行为已经太晚了！

因此，我推荐把这个值设为1.

如果不配的后果：

默认为0

配置实例：

explicit_defaults_for_timestamp = 1

##### 12.join_buffer_size
推荐设置：

16M

作用：

系统默认大小为：512k，mac下默认大小为：256k，针对128GB，1万并发的mysql我推荐给到的值为：8~16M

对于JOIN KEY 有索引和二级索引，JOIN KEY 无索引mysql会使用到join_buffer_size，一般建议设置一个很小的 GLOBAL 值，完了在 SESSION 或者 QUERY 的基础上来做一个合适的调整。如果你拍脑袋给也个4g，我们有1000个并发，就是用掉了4T的内存。。。4T啊。。。你以为你是小型机。适当的去改变它确实可以带来一定的提速，但并不是说很多值越大越好，为什么我们设置成4m呢？我们假设我们的mysql所在的vm是128gb，一根这样的join（如果被用到）是4M，1万个也不过用掉40G,而根据官方说法，total加在一起产生的join_buffer_size不要超过你所在系统的50%.默认512k肯定是小了点，我们可以适当放宽，比如说：2M，在实际使用场景时我们发觉有这样的高频操作（要看高频出现的有意义的sql的执行计划，并确认该计划的：执行cost如： "query_cost": "1003179606.87"，它产生的cost为：0.93个G,如果它真的很高频出现在调优sql到无法调优的程度，我们会去做set session join_buffer_size = 1024 * 1024 * 1024;这样的操作。而不是在一开始的my.cnf中去分配一个暴大的值，我们这边基于128gb，1万connection的并发来说，你给个16M不算小也不算多，我推荐给到8~16M间（这是指在一开始）。

如果不配的后果：

默认的为256k

配置实例：

join_buffer_size = 16M

 

##### 13.tmp_table_size
推荐设置：

64M

作用：

如果是128gb内存的服务器，我建议是在my.cnf中设成64M

通过设置tmp_table_size选项来增加一张临时表的大小，例如做高级GROUP BY操作生成的临时表。默认系统为32M，如果当你的临时表越来越多加在一起超过了这个值，那么mysql会在系统磁盘上创建，这个值不是越多越好，也没有一个合适的值。一开始的建议为>64M，然后在运行时我们通过以下公式来做临时调优，

show global status like 'created_tmp%';

把得到的结果中的：(Created_tmp_disk_tables / Created_tmp_tables) * 100% 如果<=25%为最佳值。注意了，在生产时热设定时一定要用类似以下算法：

set global tmp_table_size=64*1024*1024而不是set global tmp_table_size=64M。

如果不配的后果：

默认为32M

配置实例：

tmp_table_size = 64M

##### 15.max_allowed_packet
推荐设置：

128M

作用：

如果你经常在应用层碰到了：Got a packet bigger than'max_allowed_packet' bytes，这时你可以使用

show variables like '%max_allowed_packet%';来查看这个值，这个值没有合适，一般如：用客户端导入数据的时候，遇到 错误代码: 1153 - Got a packet bigger than 'max_allowed_packet' bytes 终止了数据导入。这样的场景下，当MySQL客户端或mysqld服务器收到大于max_allowed_packet字节的信息包时，将发出“信息包过大”错误，并关闭连接。对于某些客户端，如果通信信息包过大，在执行查询期间，可能会遇到“丢失与MySQL服务器的连接”错误。

客户端和服务器均有自己的max_allowed_packet变量，因此，如你打算处理大的信息包，必须增加客户端和服务器上的该变量。一般情况下，服务器默认max-allowed-packet为1MB,可以通过在交换机上抓包或者是图形化分析来抓返回结果判断。一般推荐在128gb内存下设置的置为128M.也可以在运行时动态调整：set global max_allowed_packet = 128*1024*1024

如果不配的后果：

1M

配置实例：

max_allowed_packet = 128M

##### 17.interactive_timeout
推荐设置：

600

作用：

单位为s，系统默认为：28800s即8小时。如果这2个值太大，你会发觉在mysql中有大量sleep的连接，这些连接又被称为：僵尸连接，僵尸连接一多你真正要用的时候就会抛：too many connection这样的错，因此对于长久不用的连接，我们一般要使用“踢出机制”，多久对于一个活动累的sql进行踢呢？我们说如果有一个长事务，它要执行1小时，我不知道这是不是属于正常？当然如果你设了太短，说1分钟就把它踢了，还真不一定踢的对，按照我们在oracle中设置的best practice我们都会把它放到10分钟。你有一条sql连着，10分钟不用，我就把它踢了，这也算正常。但是在高并发的场景下这个timeout会缩短至3-5分钟，这就是为什么我提倡我们的非报表即时类查询需要优化到sql的运行时间不超过300ms的原因，因为在高并发场景下，超过500ms的sql都已经很夸张了。保守点我觉得可以设成10分钏，在应用端由其通过jdbc连接数据库的，做的好的应用都会在jdbc里有一个autoconnect参数，这个autoconnect参数就要和mysql中的wait_timeout来做匹配了。同时在应用端要有相应的validate sql一类的操作来keep alived。不过我更推荐使用”连接池内连接的生存周期（idleConnectionTestPeriod）”来做设置，把这个置设成<mysql内的这两个值将会是最好，同时，idleConnectionTestPeriod会使用到异步的方式去做超时check。如c3p0中的：idleConnectionTestPeriod和testConnectionOnCheckin相当可靠

interactive_timeout：交互式连接超时时间(mysql工具、mysqldump等)

wait_timeout：非交互式连接超时时间，默认的连接mysql api程序,jdbc连接数据库等

interactive_timeout针对交互式连接，wait_timeout针对非交互式连接。所谓的交互式连接，即在mysql_real_connect()函数中使用了CLIENT_INTERACTIVE选项。

show global variables like 'wait_timeout';

1 timeout 只是针对空闲会话有影响。

2 session级别的wait_timeout继承global级别的interactive_timeout的值。而global级别的session则不受interactive_timeout的影响。

3 交互式会话的timeout时间受global级别的interactive_timeout影响。因此要修改非交互模式下的timeout，必须同时修改interactive_timeout的值。

4 非交互模式下，wait_timeout参数继承global级别的wait_timeout。

如果不配的后果：

系统默认为28800

配置实例：

interactive_timeout = 600

##### 18.wait_timeout
同interactive_timeout，两个值都设成一样。

##### 20.read_rnd_buffer_size
推荐设置：

8388608

作用：

就是当数据块的读取需要满足一定的顺序的情况下，MySQL 就需要产生随机读取，进而使用到 read_rnd_buffer_size 参数所设置的内存缓冲区。它的默认为256k，最大可以设到2G，它会对order by关键字起作用，当order by的计划成本超出了sort_buffer_size后，mysql会产用随机读取并消耗额外的内容，很多外面的博客说它是只对myisam引擎起作用，但其实不是，该参数还真的覆盖到所有引擎，一般它的推荐设置在8-16M，我推荐8M，根据sql分析计划如果碰到高频的查询且order by的返回包体都很大，那么再在session级别去放。

如果不配的后果：

默认为256k

配置实例：

read_rnd_buffer_size = 8M

##### 21.sort_buffer_size
推荐设置：

16M

作用：

每个会话执行排序操作所分配的内存大小。想要增大max_sort_length参数，需要增大sort_buffer_size参数。如果在SHOW GLOBAL STATUS输出结果中看到每秒输出的Sort_merge_passes状态参数很大，可以考虑增大sort_buffer_size这个值来提高ORDER BY 和 GROUP BY的处理速度。建议设置为1~4MB。当个别会话需要执行大的排序操作时，在会话级别增大这个参数。所谓会话级别，我举个例子，你拍脑袋一下，说我设个32M,你所它乘10,000请求，这得多大内存。另外，千万要注意，在mysql内存，当你的sort_buffer_size在超过2K时在底层使用的是mmap()的c函数去做内存分配的，而不是malloc()，做过c的都知道mmap()是一个矢量单位，因此它会付出性能的影响，能影响多少呢？单条sql影响值在30%。

如果不配的后果：

默认值为1M

配置实例：

sort_buffer_size =16M

##### 23.innodb_buffer_pool_size
推荐设置：

宿主机内存70%

作用：

这个值和innodb_buffer_pool_instances相辅相成。在64位机器上，这个值为8-64.

pool_instances其实为cpu核数，它的作用是：

1）对于缓冲池在数千兆字节范围内的系统，通过减少争用不同线程对缓存页面进行读写的争用，将缓冲池划分为多个单独的实例可以提高并发性。

2）使用散列函数将存储在缓冲池中或从缓冲池读取的每个页面随机分配给其中一个缓冲池实例。每个缓冲池管理自己的空闲列表， 刷新列表， LRU和连接到缓冲池的所有其他数据结构，并受其自己的缓冲池互斥量保护。

innodb_buffer_pool_size的设置需要为pool_instance的整数倍。

##### 28.innodb_lock_wait_timeout
推荐设置：

60

作用：

我们一般会碰到，mysql innodb_lock_wait_timeout这个错，这个错是慢sql导致，它代表的是慢sql的事务锁超过了mysql锁超时的设置了。默认这个值为：50s，这个值是可以动态改变的，我不建议去改这个值，因为一个sql能达50s这得多夸张？

动态改变命令如下：

SHOW GLOBAL VARIABLES LIKE 'innodb_lock_wait_timeout';

SET GLOBAL innodb_lock_wait_timeout=500;

SHOW GLOBAL VARIABLES LIKE 'innodb_lock_wait_timeout';

把它设成60s足够了。

如果不配的后果：

默认为50s

配置实例：

innodb_lock_wait_timeout = 60

##### 29.innodb_io_capacity_max
推荐设置：

8000

作用：

这个值很重要，它对读无效，对写很有决定意义。

它会直接决定mysql的tps（吞吐性能），这边给出参考：sata/sas硬盘这个值在200. sas raid10: 2000，ssd硬盘：8000， fusion-io（闪存卡）：25,000-50,000

本调优基于的是ssd，此值设置为8000，笔者上一家公司互联网金融是把一整个mysql扔到了闪存卡里的，因此设置的值为：50,000.

需要根据paas或者是ias的vm的硬盘性号来定

如果不配的后果：

默认为200，系统吞吐上不去。

配置实例：

innodb_io_capacity_max = 8000

##### 30.innodb_io_capacity
它是io_capacity_max的一半，同样，它对读无效对写有决定意义。

配置实例：

innodb_io_capacity_max = 4000

##### 31.innodb_flush_method
推荐设置：

O_DIRECT

作用：

推荐使用O_DIRECT。让我们一起来理解一下，它有3种模式：

1）fdatasync，上面最常提到的fsync(int fd)函数，该函数作用是flush时将与fd文件描述符所指文件有关的buffer刷写到磁盘，并且flush完元数据信息(比如修改日期、创建日期等)才算flush成功。它对磁盘的io读写会很频繁

2）O_DIRECT则表示我们的write操作是从mysql innodb buffer里直接向磁盘上写，它会充分利用缓存

3）_DIRECT模式的free内存下降比较慢，因为它是据文件的写入操作是直接从mysql innodb buffer到磁盘的，并不用通过操作系统的缓冲，而真正的完成也是在flush这步,日志还是要经过OS缓冲，O_DIRECT在SQL吞吐能力上较好。

如果不配的后果：

它的默认值为fdatasync。

配置实例：

innodb_flush_method = O_DIRECT

##### 39.innodb_log_file_size
推荐设置：

第1步：show engine innodb status;

得到：

Log sequence number 2944118284
Log flushed up to 2944118283
Last checkpoint at 2724318261
第2步：设innodb_log_file_size$=log Log sequence number-last checkpoint at=select (2944118284-2724318261)/1024/1024;=209M

第3步：设真正的innodb_log_file_size<=(innodb_log_files_in_group*innodb_log_file_size)*0.75,innodb_log_files_in_group为2（默认），得：

第4步：select 209/(2*0.75);=139.33即：139m，此时可把这个值设为140M

作用：

这个值的默认为5M，是远远不够的，在安装完mysql时需要尽快的修改这个值。

如果对 Innodb 数据表有大量的写入操作，那么选择合适的 innodb_log_file_size 值对提升MySQL性能很重要。然而设置太大了，就会增加恢复的时间，因此在MySQL崩溃或者突然断电等情况会令MySQL服务器花很长时间来恢复。

而这个值是没有一个绝对的概念的，MySQL的InnoDB 存储引擎使用一个指定大小的Redo log空间（一个环形的数据结构）。Redo log的空间通过innodb_log_file_size和innodb_log_files_in_group（默认2）参数来调节。将这俩参数相乘即可得到总的可用Redo log 空间。尽管技术上并不关心你是通过innodb_log_file_size还是innodb_log_files_in_group来调整Redo log空间，不过多数情况下还是通过innodb_log_file_size 来调节。为InnoDB引擎设置合适的Redo log空间对于写敏感的工作负载来说是非常重要的。然而，这项工作是要做出权衡的。你配置的Redo空间越大，InnoDB就能更好的优化写操作；然而，增大Redo空间也意味着更长的恢复时间当出现崩溃或掉电等意外时。我们是通过“测试”得到，怎么测试下面给出方法论：一般情况下我们可以按照每1GB的Redo log的恢复时间大约在5分钟左右来估算。如果恢复时间对于你的使用环境来说很重要，我建议你做一些模拟测试，在正常工作负载下（预热完毕后）模拟系统崩溃，来评估更准确的恢复时间。你可以安装 Percona Monitoring and Management，在该pmm的percona monitoring and management图表中，主要看：

1）Uncheckpointed Bytes ，如果它已经非常接近 Max Checkpoint Age，那么你几乎可以确定当前的 innodb_log_file_size 值因为太小已经某种程度上限制了系统性能。增加该值可以较为显著的提升系统性能。

2）Uncheckpointed Bytes 远小于 Max Checkpoint Age，这种情况下再增加 innodb_log_file_size 就不会有明显性能提升。

在调整完log_file_size后我们再到pmm中去看：Redo Log空间指标，比如说我们看到了1小时内有60g数据被写入日志文件，差不多就是每10分钟会有10g数据在进行“写日志“，我们需要牢牢记得，这个”写日日土已“的时间拖得越久、出现的频次越少就越有助于mysql的innodb的性能。因此这个值没有绝对推荐。如果你没有pmm，那么我们来人肉算，在上面我已经给出了人肉算的详细例子！

如果不配的后果：

默认是5M，这是肯定不够的。

配置实例：

innodb_log_file_size = 1G

##### 40.innodb_log_buffer_size
推荐设置：

16777216

作用：

对于较小的innodb_buffer_pool_size，我们会把它设成和innodb_buffer_pool_size一样。

而当超过4gb的innodb_buffer_pool_size时，我们的建议是把它切的够碎，这是mysql5.7里新带的特性，它的默认在8m，但是对于大量有事务操作的mysql我们推荐在写操作库上设置：32m

此参数确定些日志文件所用的内存大小，以M为单位。缓冲区更大能提高性能，但意外的故障将会丢失数据.官方的方档建议设置为1－8M之间！

如果不配的后果：

默认是8M

配置实例：

innodb_log_buffer_size = 32M

##### 42.innodb_large_prefix
推荐设置：

1

作用：

如果你的客户端和服务端的字符集设成了utf8mb4，那么我们需要把这个开关开启，为什么呢？mysql在5.6之前一直都是单列索引限制767，起因是256×3-1。这个3是字符最大占用空间（utf8）。但是在5.6以后，开始支持4个字节的uutf8。255×4>767, 于是增加了这个参数。这个参数默认值是OFF。当改为ON时，允许列索引最大达到3072.

在mysql5.6中这个开关叫on, off。而在5.7中叫0和1，由于我们前面设置了utf8mb4，因此这边我们必须把这个参数开启。

如果不配的后果：

不配会有问题，特别是索引会无效、或者不是走最优计划，如果你的字符集是utf8mb4，那么这个值必开启。

配置实例：

innodb_large_prefix = 1

##### 43.innodb_thread_concurrency
推荐设置：

装mysql的服务器的cpu的核数

作用：

如：64核cpu，那么推荐：64（<=cpu核数）

如果一个工作负载中，并发用户线程的数量小于等于64，建议设置innodb_thread_concurrency=0;而事实上我们的系统是处于大并发大事务的情况下的，怎么来算这个值？建议是先设置为128，然后我们不断的降这个值，直到发现能够提供最佳性能的线程数。为了安全起间我们会把它设成和cpu一样大小。

如果不配的后果：

默认是8

配置实例：

innodb_thread_concurrency = 64

##### 44.innodb_print_all_deadlocks
推荐设置：

1

作用：

推荐：1

当mysql 数据库发生死锁时， innodb status 里面会记录最后一次死锁的相关信息，但mysql 错误日志里面不会记录死锁相关信息，要想记录，启动 innodb_print_all_deadlocks 参数 。

如果不配的后果：

不会记录该信息。

配置实例：

innodb_print_all_deadlocks = 1

##### 45.innodb_strict_mode
推荐设置：

1
作用：

必须开启，没得选择，1，为什么？

从MySQL5.5.X版本开始，你可以开启InnoDB严格检查模式，尤其采用了页数据压缩功能后，最好是开启该功能。开启此功能后，当创建表（CREATE TABLE）、更改表（ALTER TABLE）和创建索引（CREATE INDEX）语句时，如果写法有错误，不会有警告信息，而是直接抛出错误，这样就可直接将问题扼杀在摇篮里。

如果不配的后果：

如果不配碰到开发或者非专业的dba会把旧ddl语句生效在5.7内，另外一个问题就是ddl语句出错时报错不明显，这会影响到“主从复制”，至于dll为什么会影响到主从复制，我们后面会在“slave_skip_errors = ddl_exist_errors”中详细解说。
配置实例：

innodb_strict_mode = 1

##### 46.log_error
error log所在位置，这个不用多讲，可以和mysql log放在同一路径下，文件名能够和其它log区分开来。

##### 47.slow_query_log
建议开启

##### 48.slow_query_log_file
慢sql所在位置，这个不用多讲，可以和mysql log放在同一路径下，文件名能够和其它log区分开来。

##### 49.log_queries_not_using_indexes=1
强烈建议开启成1.

##### 50.log_slow_admin_statements = 1
强烈建议开启成1.

##### 51.log_slow_slave_statements = 1
强烈建议开启成1.

##### 52.log_throttle_queries_not_using_indexes
推荐设置：

在一开始上线后的初期我们会开成30～50条。随着性能逐渐优化我们会把这个数量开成10.
作用：

上线前一段时间会不太稳定，我们发生过近几十条sql没有走index
如果不配的后果：

不配不开启，建议开启。

配置实例：

log_throttle_queries_not_using_indexes = 50

##### 53.expire_logs_days 
推荐设置：

30
作用：

这个值不能太大，因为你不是土豪，不能让binlog无限占用你的磁盘空间，记得这个值一旦设小，你需要做好binlog备份策略，30这个值就是30天，前提是你的binlog的备份做的有效且不占用mysql的磁盘空间。
如果不配的后果：

默认是0，即永不过期。
配置实例：

expire_logs_days = 30

##### 54.long_query_time
推荐设置：

1
作用：

默认为10秒种，即一切>=10s的sql都会被记录。我建议在开始刚上线期设成10（用默认值），越着慢sql调优越来越好，可以把这个值设成1.因为秒数越低，记录的sql越多，记录越多，也会造成mysql过慢。另外不能完全依赖于mysql的慢sql log，而是应该布署druid sql实时查看器或者是apm或者是专业的慢sql实时查询器。
如果不配的后果：

默认为10
配置实例：

long_query_time = 1

##### 58.log_bin = bin.log
主从复制时用，主从复制下的bin.log日志所在文件夹。

##### 59.sync_binlog
推荐设置：

1
作用：

主从复制时用，这个值是要看业务的，它可以有0，1，非零共3种设置方式。

1）0-代表mysql不控制写binlog的时间，由file system自由去控制，此时的mysql的并发性达到最好，但是一旦系统崩溃你会丢失很多还会写入binlog的数据（比如说你正在删数据和更新数据）

2）1-最安全，你最多丢掉一个事务或者是一条语句，但是此时它的性能很差，此参数设为0或者是1之间的性能能差4～5倍。

3）如果你用的是万兆光纤高速磁盘像或者是ssd同时data和binlog都放在一个目录下的同时你要为了安全可以开启成1.

如果不配的后果：

默认为0
配置实例：

sync_binlog = 1

##### 60.gtid_mode
推荐设置：

on
作用：

主从复制时用，推荐开启成on，它的用处就是允许你在从库上进行”备份“，从库上在进行备份时它能够获取主库的binlog位点。

该参数也可以动态在线设定。如果你要在线运行时设定，在my.cnf文件中必须把它设成on。在开启该参数时，log-bin和log-slave-updates也必须开启，否则MySQL Server拒绝启动，当开启GTID模式时，集群中的全部MySQL Server必须同时配置gtid_mod = ON，否则无法同步。


如果不配的后果：

默认为off
配置实例：

gtid_mode = on

##### 63.binlog_format

推荐设置：

row
作用：

主从复制时用，mysql5.7有3种bin log模式：

1. STATEMENT：历史悠久，技术成熟,binlog文件较小,binlog中包含了所有数据库更改信息，可以据此来审核数据库的安全等情况。binlog可以用于实时的还原，而不仅仅用于复制主从版本可以不一样，从服务器版本可以比主服务器版本高。缺点是：不是所有的UPDATE语句都能被复制，尤其是包含不确定操作的时候。调用具有不确定因素的 UDF 时复制也可能出问题，使用以下函数的语句也无法被复制：

* LOAD_FILE()

* UUID()

* USER()

* FOUND_ROWS()

* SYSDATE() (除非启动时启用了 --sysdate-is-now 选项)

同时，INSERT ... SELECT 会产生比 ROW 更多的行级锁，复制需要进行全表扫描(WHERE 语句中没有使用到索引)的 UPDATE 时，需要比 RBR 请求更多的行级锁

对于有 AUTO_INCREMENT 字段的 InnoDB表而言，INSERT 语句会阻塞其他 INSERT 语句，对于一些复杂的语句，在从服务器上的耗资源情况会更严重，而 RBR 模式下，只会对那个发生变化的记录产生影响，存储函数(不是存储过程)在被调用的同时也会执行一次 NOW() 函数，这个可以说是坏事也可能是好事，确定了的 UDF 也需要在从服务器上执行，数据表必须几乎和主服务器保持一致才行，否则可能会导致复制出错，执行复杂语句如果出错的话，会消耗更多资源。

2. ROW：任何情况都可以被复制，这对复制来说是最安全可靠的，和其他大多数数据库系统的复制技术一样。多数情况下，从服务器上的表如果有主键的话，复制就会快了很多。复制以下几种语句时的行锁更少：

* INSERT ... SELECT

* 包含 AUTO_INCREMENT 字段的 INSERT

* 没有附带条件或者并没有修改很多记录的 UPDATE 或 DELETE 语句

执行 INSERT，UPDATE，DELETE 语句时锁更少，从服务器上采用多线程来执行复制成为可能，它的缺点是：inlog 大了很多，复杂的回滚时 binlog 中会包含大量的数据，主服务器上执行 UPDATE 语句时，所有发生变化的记录都会写到 binlog 中，而 SBR 只会写一次，这会导致频繁发生 binlog 的并发写问题，UDF 产生的大 BLOB 值会导致复制变慢，无法从 binlog 中看到都复制了写什么语句。

从安全和稳定性的缩合考虑上来说我们选择ROW模式。

3. 混合式-不推荐
如果不配的后果：

5.7.6之前默认为STATEMENT模式。MySQL 5.7.7之后默认为ROW模式
配置实例：

binlog_format = row

##### 64.relay_log
主从复制用，定义relay_log的位置和名称，如果值为空，则默认位置在数据文件的目录（datadir），文件名为host_name-relay-bin.nnnnnn（By default, relay log file names have the form host_name-relay-bin.nnnnnn in the data directory）

##### 65.relay_log_recovery
推荐设置：

1
作用：

主从复制用，推荐值为1，建议打开。

当slave从库宕机后，假如relay-log损坏了，导致一部分中继日志没有处理，则自动放弃所有未执行的relay-log，并且重新从master上获取日志，这样就保证了relay-log的完整性。默认情况下该功能是关闭的，将relay_log_recovery的值设置为 1时，可在slave从库上开启该功能，建议开启。

如果不配的后果：

默认情况下是关闭的。
配置实例：

relay_log_recovery = 1

##### 66.slave_skip_errors
推荐设置：

ddl_exist_errors
作用：

主从复制用，推荐值：ddl_exist_errors。理论上我们不应该设置这个值的。即它在my.cnf文件中应该是消失的或者是这样的表示的：

#slave_skip_errors = ddl_exist_errors

但是有时我们的一些表（特别是不熟悉mysql的一些开发）真的是用的是mysql5.6旧版的建表语句，这个问题在平时单机模式下很难发现，一旦主从结构一上后，在5.7上真的是有一定机率（有10%-20%的机率）碰到ddl语句是旧版mysql而运行在mysql5.7上，这时在主从复制时会抛一个无法主从复制的错，那么这时我们需要抓数据，表已经建好了，这个影响不大、微乎其微，因此我们可以把它设成”忽略“。这个是本人的吐血经验，为什么要提这个梗。。。你们懂的。

如果不配的后果：

如果因为建表语句和mysql5.7有冲突时在单实例模式下mysql运行时不会发现，在主从复制时如果没有设跳过值，一旦发生，会影响主从复制，表现就是：主从复制失败。
配置实例：

slave_skip_errors = ddl_exist_errors

##### 70.innodb_max_undo_log_size
推荐设置：

推荐在默认值的2倍（默认为1GB）
作用：

推荐在默认值的2倍（默认为1GB），一般我们不会轻易去设它。

这个值和innodb_undo_tablespaces、innodb_undo_logs以及innodb_purge_rseg_truncate_frequency有关，这4个值是互相有牵连的。

1）innodb_undo_tablespaces必须为>=3

2）innodb_undo_logs必须开启

3）innodb_purge_rseg_truncate_frequence必须开启

如果不配的后果：

系统按照1GB来计算。
配置实例：

innodb_max_undo_log_size=2G

##### 71.innodb_purge_rseg_truncate_frequency
推荐设置：

128
作用：

默认值在128，这个值不太会去碰。控制回收undo log的频率。 指定purge操作被唤起多少次之后才释放rollback segments。当undo表空间里面的rollback segments被释放时，undo表空间才会被truncate。由此可见，该参数越小，undo表空间被尝试truncate的频率越高。
如果不配的后果：

系统默认按照：128去设定。
配置实例：

innodb_purge_rseg_truncate_frequency=128

##### 72.binlog_gtid_simple_recovery
推荐设置：

建议开启
作用：

前提是你的mysql必须>5.7.6，否则要设为关闭。

这个参数控制了当mysql启动或重启时，mysql在搜寻GTIDs时是如何迭代使用binlog文件的。

这个选项设置为真，会提升mysql执行恢复的性能。因为这样mysql-server启动和binlog日志清理更快。该参数为真时，mysql-server只需打开最老的和最新的这2个binlog文件。

如果不配的后果：

默认为0
配置实例：

binlog_gtid_simple_recovery=1

##### 73.log_timestamps
推荐设置：

system
作用：

推荐使用:system

这个参数主要是控制错误日志、慢查询日志等日志中的显示时间。但它不会影响查询日志和慢日志写到表 (mysql.general_log, mysql.slow_log) 中的显示时间，此参数是全局的，可以动态修改。

如果不配的后果：

默认值为:UTC
配置实例：

log_timestamps=system

