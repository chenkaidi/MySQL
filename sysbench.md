# sysbench 性能测试
 
由于测试需要，需要用到sysbench这个工具。推荐简便使用。

### 安装

##### yum 安装
```
yum install sysbench
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

### 磁盘IO性能测试
磁盘I/O（fileio）基准测试可以测试系统在不同I/O负载下的性能。这对于比较不同的硬盘驱动器、不同的RAID卡、不同的RAID模式，都很有帮助。可以根据测试结果来调整/0子系统。文件VO基准测试模拟了很多InnoDB的I/O特性。 测试的第一步是准备（prepare）阶段，生成测试用到的数据文件，生成的数据文件至少要比内存大。如果文件中的数据能完全放入内存中，则操作系统缓存大部分的数据，导致测试结果无法体现I/O密集型的工作负载。首先通过下面的命令创建一个数据集：

##### 例子1

1）生成测试文件,prepare阶段，生成需要的测试文件，完成后会在当前目录下生成很多小文件。
```
--num-threads 开启的线程 --file-total-size 总的文件大小

sysbench --test=fileio --file-num=10 --file-total-size=5G prepare 表示生成10个5G的文件
```
2）run
```
sysbench --test=fileio --file-total-size=5G --file-test-mode=rndrw --max-time=180 --max-requests=100000000 --num-threads=16 --init-rng=on --file-num=10 --file-extra-flags=direct --file-fsync-freq=0 --file-block-size=16384 run
```
3）清除测试数据
```
sysbench --test=fileio --file-num=10 --file-total-size=5G cleanup
```

##### 例子2 IO随机读测试样例

--创建10G的文件,分成4个,测试16K块大小,使用direct方式读,测试600秒(10分钟),启用64个线程,每3秒输出一次结果

prepare
```
#sysbench --test=fileio --file-num=4 --file-block-size=16384 --file-total-size=10G --file-test-mode=rndrd --file-extra-flags=direct --file-fsync-freq=0 --max-requests=0 --max-time=600 --num-threads=64 --report-interval=3 prepare       
```
随机读
```
#sysbench --test=fileio --file-num=4 --file-block-size=16384 --file-total-size=10G --file-test-mode=rndrd --file-extra-flags=direct --file-fsync-freq=0 --max-requests=0 --max-time=600 --num-threads=64 --report-interval=3 run
```
随机写
```
#sysbench --test=fileio --file-num=4 --file-block-size=16384 --file-total-size=10G --file-test-mode=rndwr --file-extra-flags=direct --file-fsync-freq=0 --max-requests=0 --max-time=600 --num-threads=64 --report-interval=3 run
```
随机读写
```
#sysbench --test=fileio --file-num=4 --file-block-size=16384 --file-total-size=10G --file-test-mode=rndrw --file-extra-flags=direct --file-fsync-freq=0 --max-requests=0 --max-time=600 --num-threads=64 --report-interval=3 run
```
cleanup
```
#sysbench --test=fileio --file-num=4 --file-block-size=16384 --file-total-size=10G --file-test-mode=rndrd --file-extra-flags=direct --max-requests=0 --max-time=600 --num-threads=64 --report-interval=3 cleanup
```
##### 例子3

prepare阶段，生成需要的测试文件，完成后会在当前目录下生成很多小文件。
```
sysbench --test=fileio --num-threads=16 --file-total-size=2G --file-test-mode=rndrw prepare
```
run阶段
```
sysbench --test=fileio --num-threads=20 --file-total-size=2G --file-test-mode=rndrw run
```
清理测试时生成的文件
```
sysbench --test=fileio --num-threads=20 --file-total-size=2G --file-test-mode=rndrw cleanup
```


### 数据库测试

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
sysbench --test=/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --oltp-table-size=1000000 --mysql-table-engine=innodb --oltp-tables-count=10 --mysql-user=sbtest --mysql-password=sbtestpwd --mysql-port=3306 --mysql-host=127.0.0.1 --max-requests=0 --time=3600 --report-interval=5 --threads=2 --oltp-test-mode=complex   --oltp-read-only=off run
[root@dba_test_001 scripts]# 
```

##### 清理数据
```
[root@dba_test_001 scripts]# cat 2cleanup_sysbench.sh 
#!/bin/bash
sysbench --test=/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --oltp-table-size=1000000 --mysql-table-engine=innodb --oltp-tables-count=10 --mysql-user=sbtest --mysql-password=sbtestpwd --mysql-port=3306 --mysql-host=127.0.0.1 --max-requests=0 --time=3600 --report-interval=5 --threads=2 --oltp-test-mode=complex   --oltp-read-only=off cleanup
[root@dba_test_001 scripts]# 
```
