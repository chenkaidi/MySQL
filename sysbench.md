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



##### 例子1 IO随机读测试样例

--创建10G的文件,分成4个,测试16K块大小,使用direct方式读,测试600秒(10分钟),启用64个线程,每3秒输出一次结果

prepare
```
#sysbench --test=fileio --file-num=4 --file-block-size=16384 --file-total-size=10G --file-test-mode=rndrd --file-extra-flags=direct --file-fsync-freq=0 --max-requests=0 --max-time=600 --num-threads=64 --report-interval=3 prepare       
```
```
[root@db1 ~]# ll
-rw------- 1 root root 2684354560 Jan 24 19:05 test_file.0
-rw------- 1 root root 2684354560 Jan 24 19:05 test_file.1
-rw------- 1 root root 2684354560 Jan 24 19:05 test_file.2
-rw------- 1 root root 2684354560 Jan 24 19:05 test_file.3
```

随机读
```
#sysbench --test=fileio --file-num=4 --file-block-size=16384 --file-total-size=10G --file-test-mode=rndrd --file-extra-flags=direct --file-fsync-freq=0 --max-requests=0 --max-time=600 --num-threads=64 --report-interval=3 run
```
```
Threads started!

[ 3s ] reads: 106.42 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 14.995
[ 6s ] reads: 93.69 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 15.268
[ 9s ] reads: 93.79 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 15.268
[ 12s ] reads: 93.77 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 15.268
[ 15s ] reads: 93.76 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 15.268
[ 18s ] reads: 93.71 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 14.995
[ 21s ] reads: 93.70 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 15.268
[ 24s ] reads: 93.75 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 15.268
[ 27s ] reads: 93.77 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 15.268
[ 30s ] reads: 93.71 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 15.268
[ 33s ] reads: 93.80 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 15.268
```
```
#iostat -mx 3
```
```
Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00 6000.33    0.00    93.76     0.00    32.00    63.97   10.66   10.66    0.00   0.17 100.00
scd0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
```
随机写
```
#sysbench --test=fileio --file-num=4 --file-block-size=16384 --file-total-size=10G --file-test-mode=rndwr --file-extra-flags=direct --file-fsync-freq=0 --max-requests=0 --max-time=600 --num-threads=64 --report-interval=3 run
```
```
Threads started!

[ 3s ] reads: 0.00 MiB/s writes: 54.41 MiB/s fsyncs: 0.00/s latency (ms,95%): 59.993
[ 6s ] reads: 0.00 MiB/s writes: 54.08 MiB/s fsyncs: 0.00/s latency (ms,95%): 70.548
[ 9s ] reads: 0.00 MiB/s writes: 45.10 MiB/s fsyncs: 0.00/s latency (ms,95%): 70.548
[ 12s ] reads: 0.00 MiB/s writes: 47.42 MiB/s fsyncs: 0.00/s latency (ms,95%): 62.193
[ 15s ] reads: 0.00 MiB/s writes: 50.56 MiB/s fsyncs: 0.00/s latency (ms,95%): 63.323
[ 18s ] reads: 0.00 MiB/s writes: 55.14 MiB/s fsyncs: 0.00/s latency (ms,95%): 51.945
[ 21s ] reads: 0.00 MiB/s writes: 47.95 MiB/s fsyncs: 0.00/s latency (ms,95%): 84.467
[ 24s ] reads: 0.00 MiB/s writes: 41.30 MiB/s fsyncs: 0.00/s latency (ms,95%): 81.479
[ 27s ] reads: 0.00 MiB/s writes: 51.61 MiB/s fsyncs: 0.00/s latency (ms,95%): 73.135
[ 30s ] reads: 0.00 MiB/s writes: 49.16 MiB/s fsyncs: 0.00/s latency (ms,95%): 66.838
```
```
#iostat -mx 3
```
```
Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.00 3011.00     0.00    46.74    31.79     3.83    1.27    0.00    1.27   0.33 100.00
scd0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
```

随机读写
```
#sysbench --test=fileio --file-num=4 --file-block-size=16384 --file-total-size=10G --file-test-mode=rndrw --file-extra-flags=direct --file-fsync-freq=0 --max-requests=0 --max-time=600 --num-threads=64 --report-interval=3 run
```
```
Threads started!

[ 3s ] reads: 56.65 MiB/s writes: 37.90 MiB/s fsyncs: 0.00/s latency (ms,95%): 30.815
[ 6s ] reads: 71.82 MiB/s writes: 47.70 MiB/s fsyncs: 0.00/s latency (ms,95%): 23.948
[ 9s ] reads: 68.48 MiB/s writes: 45.73 MiB/s fsyncs: 0.00/s latency (ms,95%): 25.737
[ 12s ] reads: 65.02 MiB/s writes: 43.36 MiB/s fsyncs: 0.00/s latency (ms,95%): 26.205
[ 15s ] reads: 61.05 MiB/s writes: 40.83 MiB/s fsyncs: 0.00/s latency (ms,95%): 27.659
[ 18s ] reads: 62.96 MiB/s writes: 41.94 MiB/s fsyncs: 0.00/s latency (ms,95%): 28.673
[ 21s ] reads: 69.34 MiB/s writes: 46.25 MiB/s fsyncs: 0.00/s latency (ms,95%): 24.827
[ 24s ] reads: 55.78 MiB/s writes: 37.06 MiB/s fsyncs: 0.00/s latency (ms,95%): 31.375
[ 27s ] reads: 52.08 MiB/s writes: 34.73 MiB/s fsyncs: 0.00/s latency (ms,95%): 34.330
[ 30s ] reads: 54.20 MiB/s writes: 36.07 MiB/s fsyncs: 0.00/s latency (ms,95%): 31.945
```
```
#iostat -mx 3
```
```
Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     3.33 3484.33 2320.33    54.44    36.26    32.00     6.24    1.07    0.82    1.45   0.17 100.00
scd0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
```
cleanup
```
#sysbench --test=fileio --file-num=4 --file-block-size=16384 --file-total-size=10G --file-test-mode=rndrd --file-extra-flags=direct --max-requests=0 --max-time=600 --num-threads=64 --report-interval=3 cleanup
```
##### 例子2
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
#sysbench --test=/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua  --oltp-table-size=1000000 --mysql-table-engine=innodb --oltp-tables-count=10 --mysql-user=sbtest --mysql-password=sbtestpwd --mysql-port=3306 --mysql-host=127.0.0.1 --max-requests=0 --time=10 --report-interval=1 --threads=10 --oltp-point-selects=1 --oltp-simple-ranges=0 --oltp_sum_ranges=0 --oltp_order_ranges=0 --oltp_distinct_ranges=0 --oltp-read-only=on prepare
```

##### 运行
--threads=2
```
#sysbench --test=/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --oltp-table-size=1000000 --mysql-table-engine=innodb --oltp-tables-count=10 --mysql-user=sbtest --mysql-password=sbtestpwd --mysql-port=3306 --mysql-host=127.0.0.1 --max-requests=0 --time=3600 --report-interval=5 --threads=2 --oltp-test-mode=complex   --oltp-read-only=off run
```
```
Threads started!

[ 5s ] thds: 2 tps: 168.74 qps: 3380.90 (r/w/o: 2367.23/675.78/337.89) lat (ms,95%): 20.00 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 2 tps: 167.20 qps: 3343.28 (r/w/o: 2340.05/668.82/334.41) lat (ms,95%): 19.29 err/s: 0.00 reconn/s: 0.00
[ 15s ] thds: 2 tps: 171.60 qps: 3433.38 (r/w/o: 2403.79/686.40/343.20) lat (ms,95%): 19.29 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 2 tps: 168.60 qps: 3372.41 (r/w/o: 2360.61/674.60/337.20) lat (ms,95%): 20.74 err/s: 0.00 reconn/s: 0.00
[ 25s ] thds: 2 tps: 172.00 qps: 3439.60 (r/w/o: 2407.80/687.80/344.00) lat (ms,95%): 20.00 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 2 tps: 183.60 qps: 3671.60 (r/w/o: 2570.00/734.40/367.20) lat (ms,95%): 16.41 err/s: 0.00 reconn/s: 0.00
```

--threads=10
```
#sysbench --test=/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --oltp-table-size=1000000 --mysql-table-engine=innodb --oltp-tables-count=10 --mysql-user=sbtest --mysql-password=sbtestpwd --mysql-port=3306 --mysql-host=127.0.0.1 --max-requests=0 --time=3600 --report-interval=5 --threads=10 --oltp-test-mode=complex   --oltp-read-only=off run
```
```
Threads started!

[ 5s ] thds: 10 tps: 390.04 qps: 7831.92 (r/w/o: 5484.90/1564.95/782.07) lat (ms,95%): 38.94 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 10 tps: 357.00 qps: 7135.93 (r/w/o: 4997.95/1423.99/713.99) lat (ms,95%): 42.61 err/s: 0.00 reconn/s: 0.00
[ 15s ] thds: 10 tps: 402.83 qps: 8065.48 (r/w/o: 5642.87/1616.94/805.67) lat (ms,95%): 36.24 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 10 tps: 398.83 qps: 7973.48 (r/w/o: 5580.48/1595.34/797.67) lat (ms,95%): 36.89 err/s: 0.00 reconn/s: 0.00
[ 25s ] thds: 10 tps: 376.21 qps: 7529.32 (r/w/o: 5270.48/1506.42/752.41) lat (ms,95%): 42.61 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 10 tps: 402.99 qps: 8046.75 (r/w/o: 5634.02/1606.95/805.77) lat (ms,95%): 36.24 err/s: 0.00 reconn/s: 0.00
```
--threads=50
```
#sysbench --test=/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --oltp-table-size=1000000 --mysql-table-engine=innodb --oltp-tables-count=10 --mysql-user=sbtest --mysql-password=sbtestpwd --mysql-port=3306 --mysql-host=127.0.0.1 --max-requests=0 --time=3600 --report-interval=5 --threads=50 --oltp-test-mode=complex   --oltp-read-only=off run
```
```
Threads started!

[ 5s ] thds: 50 tps: 450.36 qps: 9144.70 (r/w/o: 6425.17/1808.83/910.70) lat (ms,95%): 164.45 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 50 tps: 452.43 qps: 9053.42 (r/w/o: 6332.23/1816.32/904.86) lat (ms,95%): 179.94 err/s: 0.00 reconn/s: 0.00
[ 15s ] thds: 50 tps: 447.87 qps: 8983.49 (r/w/o: 6280.64/1807.10/895.75) lat (ms,95%): 176.73 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 50 tps: 433.64 qps: 8671.67 (r/w/o: 6070.41/1733.97/867.29) lat (ms,95%): 179.94 err/s: 0.00 reconn/s: 0.00
[ 25s ] thds: 50 tps: 431.71 qps: 8589.00 (r/w/o: 6021.94/1704.04/863.02) lat (ms,95%): 189.93 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 50 tps: 434.76 qps: 8735.12 (r/w/o: 6104.98/1760.63/869.51) lat (ms,95%): 183.21 err/s: 0.00 reconn/s: 0.00
[ 35s ] thds: 50 tps: 462.42 qps: 9195.43 (r/w/o: 6438.90/1831.28/925.24) lat (ms,95%): 173.58 err/s: 0.00 reconn/s: 0.00
```
```
[root@db1 ~]# top (按键盘“1”,显示所有CPU核)
top - 20:28:37 up 17 days,  1:49,  2 users,  load average: 11.67, 14.84, 14.66
Tasks: 102 total,   1 running, 101 sleeping,   0 stopped,   0 zombie
%Cpu0  : 83.6 us, 12.7 sy,  0.0 ni,  0.0 id,  0.3 wa,  0.0 hi,  3.3 si,  0.0 st
%Cpu1  : 86.4 us, 10.6 sy,  0.0 ni,  0.3 id,  0.0 wa,  0.0 hi,  2.7 si,  0.0 st
KiB Mem :  3880168 total,   148568 free,  2116008 used,  1615592 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  1512660 avail Mem 
```

##### 清理数据
```
[root@dba_test_001 scripts]# cat 2cleanup_sysbench.sh 
#!/bin/bash
sysbench --test=/usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --oltp-table-size=1000000 --mysql-table-engine=innodb --oltp-tables-count=10 --mysql-user=sbtest --mysql-password=sbtestpwd --mysql-port=3306 --mysql-host=127.0.0.1 --max-requests=0 --time=3600 --report-interval=5 --threads=2 --oltp-test-mode=complex   --oltp-read-only=off cleanup
[root@dba_test_001 scripts]# 
```
