# sysbench 性能测试
 
由于测试需要，需要用到sysbench这个工具。推荐简便使用。

### 安装

##### yum 安装
```
yum install sysbench
```

### 基本测试

##### 创建数据库
```
CREATE DATABASE `sbtest`  DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

GRANT ALL PRIVILEGES ON *.* TO 'sbtest'@'%' IDENTIFIED BY 'sbtestpwd';
```

##### 准备数据
```
#!/bin/bash
sysbench --test=/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua  --oltp-table-size=1000000 --mysql-table-engine=innodb --oltp-tables-count=10 --mysql-user=sbtest --mysql-password=sbtestpwd --mysql-port=3306 --mysql-host=127.0.0.1 --max-requests=0 --time=10 --report-interval=1 --threads=10 --oltp-point-selects=1 --oltp-simple-ranges=0 --oltp_sum_ranges=0 --oltp_order_ranges=0 --oltp_distinct_ranges=0 --oltp-read-only=on prepare
```

##### 运行
```
[root@dba_test_001 scripts]# cat 2run_sysbench.sh 
#!/bin/bash
sysbench --test=/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --oltp-table-size=1000000 --mysql-table-engine=innodb --oltp-tables-count=10 --mysql-user=sbtest --mysql-password=sbtestpwd --mysql-port=3306 --mysql-host=10.10.117.231 --max-requests=0 --time=3600 --report-interval=5 --threads=2 --oltp-test-mode=complex   --oltp-read-only=off run
[root@dba_test_001 scripts]# 
```

##### 清理数据
```
[root@dba_test_001 scripts]# cat 2cleanup_sysbench.sh 
#!/bin/bash
sysbench --test=/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --oltp-table-size=1000000 --mysql-table-engine=innodb --oltp-tables-count=10 --mysql-user=sbtest --mysql-password=sbtestpwd --mysql-port=3306 --mysql-host=10.10.117.231 --max-requests=0 --time=3600 --report-interval=5 --threads=2 --oltp-test-mode=complex   --oltp-read-only=off cleanup
[root@dba_test_001 scripts]# 
```


### CPU基准测试
最典型的子系统测试就是CPU基准测试。该测试使用64位整数，测试计算素数直到某个最大值所需要的时间。下面的例子将比较两台不同的GNU/Linux服务器上的测试结果。

第一台服务器配置的CPU：
```
[root@chenkaidi ~]# cat /proc/cpuinfo  |grep name
model name	: Intel(R) Xeon(R) Platinum 8163 CPU @ 2.50GHz
```
测试结果如下：
```
[root@chenkaidi ~]# sysbench --test=cpu --cpu-max-prime=20000 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.17 (using system LuaJIT 2.0.4)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 20000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:   320.32

General statistics:
    total time:                          10.0005s
    total number of events:              3204

Latency (ms):
         min:                                    2.78
         avg:                                    3.12
         max:                                  203.66
         95th percentile:                        3.02
         sum:                                 9996.93

Threads fairness:
    events (avg/stddev):           3204.0000/0.00
    execution time (avg/stddev):   9.9969/0.00
```
第二台服务器配置的CPU：
```
[root@db1 ~]# cat /proc/cpuinfo |grep name
model name	: Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz
model name	: Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz
```
测试结果如下：
```
[root@db1 ~]# sysbench --test=cpu --cpu-max-prime=20000 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.20 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 20000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:   392.65

General statistics:
    total time:                          10.0021s
    total number of events:              3928

Latency (ms):
         min:                                    2.53
         avg:                                    2.55
         max:                                    4.65
         95th percentile:                        2.57
         sum:                                 9999.73

Threads fairness:
    events (avg/stddev):           3928.0000/0.00
    execution time (avg/stddev):   9.9997/0.00

```
测试的结果简单打印出了计算出素数的时间，很容易进行比较。在上面的测试中，第二台服务器的测试结果显示比第一台快两倍。

### 文件/O基准测试
文件I/O（fileio）基准测试可以测试系统在不同I/O负载下的性能。这对于比较不同的硬盘驱动器、不同的RAID卡、不同的RAID模式，都很有帮助。可以根据测试结果来调整/0子系统。文件VO基准测试模拟了很多InnoDB的I/O特性。 测试的第一步是准备（prepare）阶段，生成测试用到的数据文件，生成的数据文件至少要比内存大。如果文件中的数据能完全放入内存中，则操作系统缓存大部分的数据，导致测试结果无法体现I/O密集型的工作负载。首先通过下面的命令创建一个数据集：
```
sysbench --test=fileio --file-total-size=10G prepare
```
这个命令会在当前工作目录下创建测试文件，后续的运行（run）阶段将通过读写这些文件进行测试。

第二步就是运行（run）阶段，针对不同的I/O类型有不同的测试选项： seqwr

顺序写入。 seqrewr

顺序重写。

seqrd

顺序读取。 rndrd

随机读取。 rndwr

随机写入。 rdnrw

混合随机读/写。 下面的命令运行文件I/O混合随机读/写基准测试：
```
sysbench --test=fileio --file-total-size=10G --file-test-mode=rndrn --init-rng=on --max-time=300 --max-requests=0 run

```
​
结果如下：
```
sysbench vo.4.8：multithreaded system evaluation benchmark Running the test with following options：Number of threads：1
Initializing random number generator from timer.
Extra file open flags：0128files，1.1719Gb each
150Cb total file size Block size 16kb Number of random requests for random I0：10000
Read/Nrite ratio for conbined random I0 test：1.50
Periodic FSYNC enabled，calling fsync（）each 100 requests.
Calling fsync（）at the end of test，Enabled.
Using synchronous I/0mode Doing random r/w test Threads started！
Time limit exceeded，exiting...
Done.
Operations performed：40260 Read，26840 Write，85785 other =152885 Total Read 629.o6Mb Written 419.38Mb Total transferred 1.0239Cb（3.4948Mb/sec）
223.67 Requests/sec executed Test execution summary：
300.00045
total number of events：67100
total time taken by event execution：254.4601
per-request statistics：min：
0.00005
max：0.56285
approx.95 percentile：0.0099s Threads fairness：events（avg/stddev）：67100.0000/0.00
execution time（avg/stddev）：254.4601/0.00
```
输出结果中包含了大量的信息。和I/0子系统密切相关的包括每秒请求数和总吞吐量。 在上述例子中，每秒请求数是223.67 Requests/sec，吞吐量是3.4948MB/sec。另外，时间信息也非常有用，尤其是大约95%的时间分布。这些数据对于评估磁盘性能十分有用。 测试完成后，运行清除（cleanup）操作删除第一步生成的测试文件：
```
sysbench --testafileio --file-total-size=10G cleanup
```
### OLTP基准测试

OLTP基准测试模拟了一个简单的事务处理系统的工作负载。下面的例子使用的是一张 超过百万行记录的表，第一步是先生成这张表：
```
$sysbench --test=oltp --oltp-table-size=1000000 --mysq1-db=test/
-mysql-user=root prepare 
```
sysbench vo.4.8：multithreaded system evaluation benchmark

No DB drivers specified，using mysql Creating tablesbtest... Creating 1000000 records in table‘sbtest... 生成测试数据只需要上面这条简单的命令即可。接下来可以运行测试，这个例子采用了 8个并发线程，只读模式，测试时长60秒：
```
$sysbench --test=oltp --oltp-table-size=1000000 --mysq1-db=test --mysql-user=root/
-max-time=60 --oltp-read-only=on --max-requests=0 --nun-threads=8 run 
```
​
```
sysbench vo.4.8：multithreaded system evaluation benchmark No DB drivers specified，using mysql WARNING:Preparing of"BEGIN"is unsupported，using emulation
（last message repeated 7 times）
Running the test with following options：Number of threads：8
Doing OLTP test.
Running mixed OLTP test Doing read-only test Using Special distribution（12 iterations，1 pct of values are returned in 75 pct cases）
Using"BECIN"for starting transactions Using auto inc on the id column Threads started！
Time limit exceeded，exiting..…
（last message repeated 7 times）
Done.
OLTP test statistics：queries performed：
179606write：
0
total：
205264
transactions：
12829（213.07 per sec.）
deadlocks：
0（o.oo per sec.）
read/write requests：179606（2982.92 per sec.）
other operations：25658（426.13 per sec.）
Test execution summary：total time：
60.21145
total number of events：12829
total time taken by event execution：480.2086
per-request statistics：max：
1.91065
approx.95 percentile：0.1163s Threads fairness：events（avg/stddev）：1603.6250/70.66
execution time（avg/stddev）：60.0261/0.06
```
如上所示，结果中包含了相当多的信息。其中最有价值的信息如下：

总的事务数。·每秒事务数。

时间统计信息（最小、平均、最大响应时间，以及95%百分比响应时间）。

线程公平性统计信息（thread-fairmess），用于表示模拟负载的公平性。

这个例子使用的是sysbench的第4版，在SourceForge.net可以下载到这个版本的编译好的可执行文件。也可以从Launchpad下载最新的第5版的源代码自行编译（这是一件简单、有用的事情），这样就可以利用很多新版本的特性，包括可以基于多个表而不是单个表进行测试，可以每隔一定的间隔比如10秒打印出吞吐量和响应的结果。这些指标对于理解系统的行为非常重要。
