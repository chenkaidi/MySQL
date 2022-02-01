##### 01.哪个SQL 执行最多：
```
select SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN from events_statements_summary_by_digest order by COUNT_STAR desc limit 1;
```
##### 02.哪个SQL 平均响应时间最多AVG_TIMER_WAIT：
```
select SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN from events_statements_summary_by_digest order by AVG_TIMER_WAIT desc limit 1;
```
##### 03.哪个SQL 扫描的行数最多：
```
select SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN from events_statements_summary_by_digest order by SUM_ROWS_EXAMINED desc limit 1;
```
##### 04.哪个SQL 使用的临时表最多：
```
select SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN from events_statements_summary_by_digest order by SUM_CREATED_TMP_DISK_TABLES desc limit 1;
```
```
select SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN from events_statements_summary_by_digest order by SUM_CREATED_TMP_TABLES desc limit 1;
```
##### 05.哪个SQL 返回的结果集最多：
```
select SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_RO WS_EXAMINED,FIRST_SEEN,LAST_SEEN from events_statements_summary_by_digest order by SUM_ROWS_SENT desc limit 1;
```
##### 06.哪个SQL 排序数最多：
```
select SCHEMA_NAME,DIGEST_TEXT,COUNT_STAR,AVG_TIMER_WAIT,SUM_ROWS_SENT,SUM_ROWS_EXAMINED,FIRST_SEEN,LAST_SEEN from events_statements_summary_by_digest order by SUM_SORT_ROWS desc limit 1;
```
##### 07.哪个表、文件逻辑IO 最多(热数据)：
```
select FILE_NAME,EVENT_NAME,COUNT_READ,SUM_NUMBER_OF_BYTES_READ,COUNT_WRITE,SUM_NUMBER_OF_BYTES_WRITE from file_summary_by_instance order by SUM_NUMBER_OF_BYTES_READ+SUM_NUMBER_OF_BYTES_WRITE desc limit 2;
```
##### 08.哪个索引使用最多：
```
select OBJECT_NAME, INDEX_NAME, COUNT_FETCH, COUNT_INSERT, COUNT_UPDATE,COUNT_DELETE from table_io_waits_summary_by_index_usage order by SUM_TIMER_WAIT desc limit 1;
```
##### 09.哪个索引没有使用过：
```
select OBJECT_SCHEMA, OBJECT_NAME, INDEX_NAME from table_io_waits_summary_by_index_usage where INDEX_NAME is not null and COUNT_STAR = 0 and OBJECT_SCHEMA <> 'mysql' order by OBJECT_SCHEMA,OBJECT_NAME;
```
##### 10.哪个等待事件消耗的时间最多：
```
select EVENT_NAME, COUNT_STAR, SUM_TIMER_WAIT, AVG_TIMER_WAIT from events_waits_summary_global_by_event_name where event_name != 'idle' order by SUM_TIMER_WAIT desc limit 1;
```
