## 案例

##### 案例背景:

```硬件及软件环境:
联想服务器（IBM） 
磁盘500G 没有raid
centos 6.8
mysql 5.6.33  innodb引擎  独立表空间
备份没有，日志也没开
```
```
开发用户专用库:
jira(bug追踪) 、 confluence(内部知识库)    ------>LNMT
```

##### 故障描述:
```
断电了，启动完成后“/” 只读
fsck  重启,系统成功启动,mysql启动不了。
结果：confulence库在  ， jira库不见了
```

##### 学员求助内容:
```
求助：
这种情况怎么恢复？
我问：
有备份没
求助：
连二进制日志都没有，没有备份，没有主从
我说：
没招了，jira需要硬盘恢复了。
求助：
1、jira问题拉倒中关村了
2、能不能暂时把confulence库先打开用着
将生产库confulence，拷贝到1:1虚拟机上/var/lib/mysql,直接访问时访问不了的

问：有没有工具能直接读取ibd
我说：我查查，最后发现没有
```

##### 我想出一个办法来：
```
表空间迁移:
create table xxx
alter table  confulence.t1 discard tablespace;
alter table confulence.t1 import tablespace;
虚拟机测试可行。
```

##### 处理问题思路:

```
confulence库中一共有107张表。
1、创建107和和原来一模一样的表。
他有2016年的历史库，我让他去他同时电脑上 mysqldump备份confulence库
mysqldump -uroot -ppassw0rd -B  confulence --no-data >test.sql
拿到你的测试库，进行恢复
到这步为止，表结构有了。
2、表空间删除。
select concat('alter table ',table_schema,'.'table_name,' discard tablespace;') from information_schema.tables where table_schema='confluence' into outfile '/tmp/discad.sql';
source /tmp/discard.sql
执行过程中发现，有20-30个表无法成功。主外键关系
很绝望，一个表一个表分析表结构，很痛苦。
set foreign_key_checks=0 跳过外键检查。
把有问题的表表空间也删掉了。
3、拷贝生产中confulence库下的所有表的ibd文件拷贝到准备好的环境中
select concat('alter table ',table_schema,'.'table_name,' import tablespace;') from information_schema.tables where table_schema='confluence' into outfile '/tmp/discad.sql';
4、验证数据
表都可以访问了，数据挽回到了出现问题时刻的状态（2-8）
```
