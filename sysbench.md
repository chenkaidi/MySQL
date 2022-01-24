# sysbench 性能测试
 

由于测试需要，需要用到sysbench这个工具。推荐简便使用。

##### yum 安装
```
yum install sysbench
```
 
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
