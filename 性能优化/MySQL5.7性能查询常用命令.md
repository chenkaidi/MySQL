### mysql5.7性能查询常用命令

##### 1,查看每个客户端IP过来的连接消耗了多少资源。
```
mysql> select * from host_summary;
```

##### 2,查看某个数据文件上发生了多少IO请求。
```
mysql> select * from io_global_by_file_by_bytes;
```

##### 3,查看每个用户消耗了多少资源。
```
mysql> select * from user_summary;
```

##### 4,查看总共分配了多少内存。
```
mysql> select * from memory_global_total;
```

##### 5,数据库连接来自哪里，以及这些连接对数据库的请求情况是怎样的？
查看当前连接情况。
mysql> select host, current_connections, statements from host_summary;

##### 6,查看当前正在执行的SQL和执行show full processlist的效果相当。
```
mysql> select conn_id, user, current_statement, last_statement from session;
```

##### 7,数据库中哪些SQL被频繁执行？
执行下面命令查询TOP 10最热SQL。
```
mysql> select db,exec_count,query from statement_analysis order by exec_count desc limit 10;
```

##### 8,哪个文件产生了最多的IO，读多，还是写的多？
```
mysql> select * from io_global_by_file_by_bytes limit 10;
```

##### 9,哪个表上的IO请求最多？
```
mysql> select * from io_global_by_file_by_bytes where file like '%ibd' order by total desc limit 10;
```

##### 10,哪个表被访问的最多？
先访问statement_analysis，根据热门SQL排序找到相应的数据表。哪些语句延迟比较严重？查看statement_analysis中avg_latency的最高的SQL。
```
mysql> select * from statement_analysis order by avg_latency desc limit 10;
```

##### 11,或者查看statements_with_runtimes_in_95th_percentile视图。
```
mysql> select * from statements_with_runtimes_in_95th_percentile;
```

##### 12,哪些SQL执行了全表扫描或执行了排序操作？
```
mysql> select * from statements_with_sorting;
mysql> select * from statements_with_full_table_scans;
```

##### 13,哪些SQL语句使用了临时表，又有哪些用到了磁盘临时表？
查看statement_analysis中哪个SQL的tmp_tables 、tmp_disk_tables值大于0即可。
```
mysql> select db, query, tmp_tables, tmp_disk_tables from statement_analysis where tmp_tables>0 or tmp_disk_tables >0 order by (tmp_tables+tmp_disk_tables) desc limit 20;
```

##### 14,也可以查看statements_with_temp_tables视图。
```
mysql> select * from statements_with_temp_tables\G
```

##### 15 哪个表占用了最多的buffer pool？
```
mysql> select * from innodb_buffer_stats_by_table order by allocated desc limit 10;
```

##### 16,每个库（database）占用多少buffer pool？
```
mysql> select * from innodb_buffer_stats_by_schema order by allocated desc limit 10;
```

##### 17,每个连接分配多少内存？
利用session表和memory_by_thread_by_current_bytes分配表进行关联查询。
```
mysql> select b.user, current_count_used, current_allocated, current_avg_alloc, current_max_alloc, total_allocated,current_statement from memory_by_thread_by_current_bytes a, session b where a.thread_id = b.thd_id;
```

##### 18,MySQL自增长字段的最大值和当前已经使用到的值？
```
mysql> select * from schema_auto_increment_columns;
```

##### 19,MySQL索引使用情况统计？
```
mysql> select * from schema_index_statistics;
```

##### 20,MySQL有哪些冗余索引和无用索引？
```
mysql> select * from schema_redundant_indexes;
mysql> select * from schema_unused_indexes;
```

##### 21,MySQL内部有多个线程在运行？
MySQL内部的线程类型及数量。
```
mysql> select user, count(*) from processlist group by user;
```
