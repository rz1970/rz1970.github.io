---
title: 初探postgres命令
tags: ["greenplum", "postgres"]
---


## 用户与角色
首先：在PostgreSQL里没有区分用户和角色的概念，`"CREATE USER"`为 `"CREATE ROLE"`的别名，这两个命令几乎是完全相同的，唯一的区别是`"CREATE USER"`命令创建的用户默认带有`LOGIN`属性，而`"CREATE ROLE"`命令创建的用户默认不带`LOGIN`属性。

帮助(Help)命令，在已经登陆postgres的情况下
```
\?                print the current help
\? [commands]     show help on backslash commands
\? options        show help on psql command-line options
\? variables      show help on special variables
\h [NAME]         help on syntax of SQL commands, * for all commands
```
1. step 1: 创建 manager 角色和jack 用户
```
testdb=# create role manager;
CREATE ROLE
testdb=# create user jack;
CREATE ROLE
```
2. step 2: 验证
```
testdb=# \du
                                                          List of roles
   Role name   |                                     Attributes                                                 | Member of
----------------+------------------------------------------------------------------------+-------------
 Ryan               | Superuser, Create role, Create DB, Replication, Bypass RLS     | {}
 jack                |                                                                                                       | {}
 manager         | Cannot login                                                                                 | {}
```
```
testdb=# SELECT rolname from pg_roles ;
    rolname
----------------
 Ryan
 manager
 jack
(3 rows)
```
角色manager创建时没有分配login权限，所以没有创建用户 
```
testdb=# SELECT usename from pg_user;
    usename
----------------
 Ryan
 jack
(2 rows)
```
```
$ psql -d testdb -Ujack;
psql (9.5.3)
Type "help" for help.
```
```
$ psql -d testdb -Umanager;
psql: FATAL:  role "manager" is not permitted to log in
```
3. step3: 修改manager的权限，增加LOGIN属性
```
testdb=# alter role manager login;
ALTER ROLE
```
```
testdb=# \du
                                      List of roles
   Role name    |                         Attributes                                                           | Member of
----------------+-----------------------------------------------------------------------+-----------
 Ryan               | Superuser, Create role, Create DB, Replication, Bypass RLS   | {}
jack                 |                                                                                                    | {}
 manager         |                                                                                                    | {}
```
给manager 角色分配login权限，系统将自动创建同名用户david
```
testdb=# select usename from pg_user;
    usename
----------------
 Ryan
 jack
 manager
(5 rows)
```

## 命令小结：
1. psql 终端可以用\du 或\du+ 查看，也可以查看系统表 select * from pg_roles;
```
testdb=# \du
testdb=# \du+
testdb=# select * from pg_roles;
```
2. 修改数据库的owner
```
testdb=# alter database testdb owner to tester
```