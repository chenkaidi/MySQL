1、安装数据库

```
#!/bin/bash

if id -u mysql >/dev/null 2>&1; then
        echo "user exists"
else
        groupadd mysql
        useradd -r -g mysql mysql
fi


if ss -nlut | grep 3306 ;then
	echo "3306 exist"
	exit 22
else
	echo "installing mysql"
fi


if  [ -d /data/mysql ] ;then 
	echo "/data/mysql  directory is exist"
	exit 23
else
	echo "mkdir /data/mysql -p"
fi


mkdir /data/mysql/my3306/logs -p
mkdir /data/mysql/my3306/dbbak -p
touch /data/mysql/my3306/logs/mysqld.log
tar -xvf /data/mysql-8.0.23-linux-glibc2.12-x86_64.tar.xz -C /data/mysql
if [ -f "$myFile" ]; then
   cp /etc/my.cnf  /etc/my.cnf-`date +%s`
fi
cat  > /etc/my.cnf  <<eof
[mysqld]
server-id = 3306
port = 3306
basedir = /data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64
datadir = /data/mysql/my3306/data
socket = /tmp/mysql.sock
secure_file_priv = /data/mysql/my3306/dbbak
log-error = /data/mysql/my3306/logs/mysqld.log
pid-file = /data/mysql/my3306/logs/mysqld.pid
long_query_time               = 1
slow_query_log                = on
log_queries_not_using_indexes = off
slow_query_log_file =/data/mysql/my3306/logs/slow.log
character_set_server = utf8
init_connect = 'SET NAMES utf8'
lower_case_table_names = 1  #不区分大小写
max_connections = 5000
default-time_zone = '+8:00'
eof

echo 'export PATH=$PATH:/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin' >> /etc/profile
source /etc/profile
chown -R mysql:mysql /data/mysql
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysqld --initialize --user=mysql --basedir=/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/ --datadir=/data/mysql/my3306/data/

Passwd=`tail -1 /data/mysql/my3306/logs/mysqld.log  | sed -nr 's@^.*root\@localhost: (.*)$@\1@p'`
echo "mysql is password:${Passwd}"

cat >  /etc/systemd/system/mysqld.service << EOF

[Unit]
Description=MySQL Server
Documentation=man:mysqld(5)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql
ExecStart=/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE = 5000
EOF


systemctl restart mysqld
systemctl enable mysqld

sleep 20

/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -uroot -p`grep "A temporary password is generated for root@localhost" /data/mysql/my3306/logs/mysqld.log |awk '{print $NF}'` --connect-expired-password -e "alter user 'root'@'localhost' identified by 'City_ops123.';"
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -uroot -p'City_ops123.' --connect-expired-password -e "flush privileges;"
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -uroot -p'City_ops123.' --connect-expired-password -e	"create user 'root'@'%' identified with mysql_native_password by 'City_ops123.';"
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -uroot -p'City_ops123.' --connect-expired-password -e	"grant all privileges on *.* to 'root'@'%' with grant option;"
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -uroot -p'City_ops123.' --connect-expired-password -e	"flush privileges;"

/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -uroot -p'City_ops123.' --connect-expired-password -e "create user 'rep01'@'%' identified with mysql_native_password by 'City_ops123.';"
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -uroot -p'City_ops123.' --connect-expired-password -e "grant replication slave on *.* to 'rep01'@'%' with grant option;"
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -uroot -p'City_ops123.' --connect-expired-password -e "flush privileges;"
```

2、设置master1

```
#!/bin/bash
cp /etc/my.cnf  /etc/my.cnf-`date +%s`
cat  > /etc/my.cnf  <<eof
[mysqld]
explicit_defaults_for_timestamp=true
port             = 3306
basedir          = /data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64
datadir          = /data/mysql/my3306/data
server_id        = 643306
socket           = /tmp/mysql.sock
back_log         = 350
connect_timeout  = 20
event_scheduler  = off
local_infile     = off
secure_file_priv = /data/mysql/my3306/dbbak
pid-file         = /data/mysql/my3306/logs/mysql.pid
wait_timeout     = 1800
interactive_timeout  =1800
character-set-client-handshake = FALSE
character_set_server = utf8mb4
collation-server = utf8mb4_unicode_ci
skip_name_resolve
log_timestamps=system
default-time_zone = '+8:00'
skip-ssl

#########  bin log      ####################
binlog_format                = row
binlog_row_image             = full
binlog_cache_size            = 64M
max_binlog_size              = 1G
max_binlog_cache_size        = 2G
binlog_rows_query_log_events = 0
master_verify_checksum       = 1
slave_allow_batching         = 1
#expire_logs_days            = 14
binlog_expire_logs_seconds	 = 1209066
log-bin                      = /data/mysql/my3306/logs/mysql_bin

######### error log    ####################
#log_warnings     = 2
log_error        = /data/mysql/my3306/logs/mysqld.log

########## general_log ####################
general_log 	= off
general_log_file = /data/mysql/my3306/logs/general.log

########## slow log    ####################
long_query_time               = 1 
slow_query_log                = on 
log_queries_not_using_indexes = off
#log_slow_admin_statements     = on
#log_slow_slave_statements     = on
slow_query_log_file = /data/mysql/my3306/logs/slow.log

##########replication####################
#skip_slave_start
read_only                 = off
slave_parallel_type       = logical_clock
slave_parallel_workers    = 4
relay_log_recovery        = on
sync_master_info          = 10000
slave_compressed_protocol = off
slave_net_timeout         = 10
log_slave_updates
relay_log= /data/mysql/my3306/logs/relay_bin

#slave_skip_errors = all
#slave-skip-errors=1064 1146 1062
#rpl_semi_sync_master_enabled = ON
#rpl_semi_sync_slave_enabled = ON
#rpl_semi_sync_master_timeout = 10000

gtid_mode                = ON
enforce_gtid_consistency = ON
relay_log_purge		= ON

#replicate_wild_ignore_table =mysql.%
#replicate_wild_ignore_table =information_schema.%
#replicate_wild_ignore_table =performance_scheme.%
#replicate_wild_ignore_table =sys.%
#replicate_wild_ignore_table =percona.%

########### innodb ####################
innodb_buffer_pool_size         = 16384M
innodb_buffer_pool_instances    = 6
innodb_log_files_in_group       = 3
innodb_log_file_size            = 256M
innodb_log_buffer_size          = 16M
innodb_lock_wait_timeout        = 40
innodb_file_per_table           = 1
innodb_stats_on_metadata        = OFF
lower_case_table_names          = 1
innodb_flush_method             = O_DIRECT
innodb_flush_log_at_trx_commit  = 1
log_bin_trust_function_creators = 1
sync_binlog                     = 1
innodb_open_files               = 1024
innodb_thread_concurrency       = 0
innodb_print_all_deadlocks      = ON
performance_schema              = ON
innodb_undo_log_truncate        = 1
#innodb_undo_tablespaces         = 3
innodb_rollback_segments        = 128
innodb_max_undo_log_size        = 1073741824

#innodb_purge_threads           = 1
##innodb_write_io_threads       = 4
##innodb_read_io_threads        = 4
#thread_pool
#thread_handling                =pool-of-threads
#thread_pool_stall_limit        =10

#########for SSD ###################
innodb_io_capacity       = 500 
innodb_adaptive_flushing = OFF
innodb_flush_neighbors   = 2

#########per_thread_buffers####################
max_connections           = 10000
max_connect_errors        = 2048
max_allowed_packet        = 100M
max_heap_table_size       = 256M
tmp_table_size            = 256M
max_prepared_stmt_count   = 1M
query_alloc_block_size    = 128K
join_buffer_size          = 512k
key_buffer_size           = 1M
query_prealloc_size       = 64K
read_buffer_size          = 1M
read_rnd_buffer_size      = 512K
sort_buffer_size          = 2M
table_open_cache          = 2048
table_open_cache_instances= 8
thread_cache_size         = 128
group_concat_max_len      = 2048
skip-external-locking
eof

systemctl restart mysqld
```

3、设置master2

```
#!/bin/bash
cp /etc/my.cnf  /etc/my.cnf-`date +%s`
cat  > /etc/my.cnf  <<eof
[mysqld]
explicit_defaults_for_timestamp=true
port             = 3306
basedir          = /data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64
datadir          = /data/mysql/my3306/data
server_id        = 653306
socket           = /tmp/mysql.sock
back_log         = 350
connect_timeout  = 20
event_scheduler  = off
local_infile     = off
secure_file_priv = /data/mysql/my3306/dbbak
pid-file         = /data/mysql/my3306/logs/mysql.pid
wait_timeout     = 1800
interactive_timeout  =1800
character-set-client-handshake = FALSE
character_set_server = utf8mb4
collation-server = utf8mb4_unicode_ci
skip_name_resolve
log_timestamps=system
default-time_zone = '+8:00'
skip-ssl

#########  bin log      ####################
binlog_format                = row
binlog_row_image             = full
binlog_cache_size            = 64M
max_binlog_size              = 1G
max_binlog_cache_size        = 2G
binlog_rows_query_log_events = 0
master_verify_checksum       = 1
slave_allow_batching         = 1
#expire_logs_days            = 14
binlog_expire_logs_seconds	 = 1209066
log-bin                      = /data/mysql/my3306/logs/mysql_bin

######### error log    ####################
#log_warnings     = 2
log_error        = /data/mysql/my3306/logs/mysqld.log

########## general_log ####################
general_log 	= off
general_log_file = /data/mysql/my3306/logs/general.log

########## slow log    ####################
long_query_time               = 1 
slow_query_log                = on 
log_queries_not_using_indexes = off
#log_slow_admin_statements     = on
#log_slow_slave_statements     = on
slow_query_log_file = /data/mysql/my3306/logs/slow.log

##########replication####################
#skip_slave_start
read_only                 = off
slave_parallel_type       = logical_clock
slave_parallel_workers    = 4
relay_log_recovery        = on
sync_master_info          = 10000
slave_compressed_protocol = off
slave_net_timeout         = 10
log_slave_updates
relay_log= /data/mysql/my3306/logs/relay_bin

#slave_skip_errors = all
#slave-skip-errors=1064 1146 1062
#rpl_semi_sync_master_enabled = ON
#rpl_semi_sync_slave_enabled = ON
#rpl_semi_sync_master_timeout = 10000

gtid_mode                = ON
enforce_gtid_consistency = ON
relay_log_purge		= ON

#replicate_wild_ignore_table =mysql.%
#replicate_wild_ignore_table =information_schema.%
#replicate_wild_ignore_table =performance_scheme.%
#replicate_wild_ignore_table =sys.%
#replicate_wild_ignore_table =percona.%

########### innodb ####################
innodb_buffer_pool_size         = 16384M
innodb_buffer_pool_instances    = 6
innodb_log_files_in_group       = 3
innodb_log_file_size            = 256M
innodb_log_buffer_size          = 16M
innodb_lock_wait_timeout        = 40
innodb_file_per_table           = 1
innodb_stats_on_metadata        = OFF
lower_case_table_names          = 1
innodb_flush_method             = O_DIRECT
innodb_flush_log_at_trx_commit  = 1
log_bin_trust_function_creators = 1
sync_binlog                     = 1
innodb_open_files               = 1024
innodb_thread_concurrency       = 0
innodb_print_all_deadlocks      = ON
performance_schema              = ON
innodb_undo_log_truncate        = 1
#innodb_undo_tablespaces         = 3
innodb_rollback_segments        = 128
innodb_max_undo_log_size        = 1073741824

#innodb_purge_threads           = 1
##innodb_write_io_threads       = 4
##innodb_read_io_threads        = 4
#thread_pool
#thread_handling                =pool-of-threads
#thread_pool_stall_limit        =10

#########for SSD ###################
innodb_io_capacity       = 500 
innodb_adaptive_flushing = OFF
innodb_flush_neighbors   = 2

#########per_thread_buffers####################
max_connections           = 10000
max_connect_errors        = 2048
max_allowed_packet        = 100M
max_heap_table_size       = 256M
tmp_table_size            = 256M
max_prepared_stmt_count   = 1M
query_alloc_block_size    = 128K
join_buffer_size          = 512k
key_buffer_size           = 1M
query_prealloc_size       = 64K
read_buffer_size          = 1M
read_rnd_buffer_size      = 512K
sort_buffer_size          = 2M
table_open_cache          = 2048
table_open_cache_instances= 8
thread_cache_size         = 128
group_concat_max_len      = 2048
skip-external-locking
eof

systemctl restart mysqld
```

4、设置master1

```
mysql_slave_ip=10.168.70.13
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -u root -p'City_ops123.' -e "change master to \
                          master_host='${mysql_slave_ip}', \
                          master_user='rep01', \
                          master_password='City_ops123.', \
                          MASTER_AUTO_POSITION=1, \
                          GET_MASTER_PUBLIC_KEY=1"
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -u root -p'City_ops123.' -e "start slave;" > /dev/null
```

5、设置master2

```
mysql_master_ip=10.168.70.12
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -u root -p'City_ops123.' -e "change master to \
                          master_host='${mysql_master_ip}', \
                          master_user='rep01', \
                          master_password='City_ops123.', \
                          MASTER_AUTO_POSITION=1, \
                          GET_MASTER_PUBLIC_KEY=1"
/data/mysql/mysql-8.0.23-linux-glibc2.12-x86_64/bin/mysql -u root -p'City_ops123.' -e "start slave;" > /dev/null
```

6配置keepalived ------和mysql5一样的配置此处省略
