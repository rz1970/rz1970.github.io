---
title: 数据管理及查询
tags: greenplum
---

# GP的并发控制

MVCC(多版本控制模型)：Mutltiversion Concurrency Control: 避免数据库事务显示锁定，最大化减少锁争用以确保多用户环境下的性能。

# 插入新记录

使用`insert`命令插入数据。从另一个表中获取并插入到当前表，可以用户创建大量的模拟数据

```sql
insert into tb_cp_02 select * from tb_cp_02 where date < '2013-12-01';
```

AO表不建议批量`insert`插入数据

# 更新记录

使用`update`更新数据。GP的DK(分布键)不可以被`update`

# 删除记录

使用`delete`命令删除记录。`truncate tablename` 快速删除表数据

# 事务管理

多个SQL语句放在一起作为一个整体操作，所有SQL一起成功或失败
GP中执行事务的SQL命令：
+ 使用`begin`或`start transaction`开启一个事务块
+ 使用`end`或`commit`提交事务块;
+ 使用`rollback`回滚事务而不提交任何修改
+ 使用`savepoint`选择性的保存事务点，之后可以使用`rollback to savepoint`回滚到之前保存的事务

事务隔离级别（SQL标准定义了4中）：
+ `已提交读`：select只能看到查询开始前的（缺省）
+ `可串行化`：事务被要求串行执行（最严格的）
+ `未提交读`：在GP中等同`已提交读`
+ `可重复读`：在GP中等同`串行化`

```sql
-- 查看事务隔离的级别
show transaction_isolation;
```

# 回收空间和分析

+ 事务ID管理：上限为400万。建议每200万时执行`vacuum`
+ 系统目录管理：大量`create`或`drop`命令会导致系统表的迅速膨胀
+ MVCC事务并发模式：已删除或更新的记录仍然占据磁盘空间
+ 过期的记录会被存放在叫做`自由空间映射`的地方
+ 超出`自由映射空间`的过期记录所占用的空间无法回收
+ `vacuum full`回收所有过期记录，但耗时长
+ 使用`create table as`来处理自由空间溢出的情况
+ 自由映射空间的设置参数：
  + max_fsm_pages: 最大页数
  + max_fsm_relations: 最大对象数
+ GP使用基于成本的查询优化器
+ `ANALYZE`命令收集查询优化器需要的统计信息

```sql
-- 空间回收
VACUUM tb01;
-- 最大页数
show max_fsm_pages
-- 最大对象数
show max_fsm_relations
-- 统计信息
VACUUM ANALYZE tb_cp_02
```

# 日常重建索引

+ 对于B-tree索引，新建比更新较多的索引要快
+ 重建索引可以回收过期的空间
+ 在GP中，删除索引然后创建通常比`reindex`更快
+ 当更新索引列时，Bitmap索引通常不会被更新

# 管理GP日志文件

+ 数据库服务日志文件
  + GP日志输出量大且不需要无期限的保存这些日志
  + Master和Segment上都开启了日志文件`按天滚动`
  + 日志文件存放在：`pg_log`目录下。格式：`gpdb-YYYY-MM-DD_time.csv`
+ 搜索数据库服务日志文件
  + `gplogfilter`: 查找匹配指定标准的日志数据
  + 默认只查找master上的日志文件
  + 通过使用`gplogfilter`+`gpssh`可以在所有节点进行查找
+ 程序日志文件
  + 缺省位于`~/gpAdminLogs`目录下
  + 命令方式：`<script_name>_<date>.log`
  + 日志记录的格式：`<timestamp>:<utility>:<host>:<user>:[info|warn|fatal]:<message>`

```sql
-- 显示master日志文件的最近三行记录
gplogfilter -n 3
-- 显示每个segment日志文件的最近三行记录
gpssh -f seg_host_file => gplogfilter -n 3 /data/primary/*/pg_log/gpdb_*.csv
```