---
title: 在ubuntu 14.04上部署postgres
tags: postgres
---

## Installation
Ubuntu's default repositories contain Postgres packages, so we can install them without a hassle using the apt packaging system.

Since we haven't updated our local apt repository lately, let's do that now. We can then get the Postgres package and a "contrib" package that adds some additional utilities and functionality:
```
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```
Now that our software is installed, we can go over how it works and how it may be different from similar database management systems you may have used.

## Using PostgreSQL Roles and Databases
By default, Postgres uses a concept called "roles" to aid in authentication and authorization. These are, in some ways, similar to regular Unix-style accounts, but Postgres does not distinguish between users and groups and instead prefers the more flexible term "role".

Upon installation Postgres is set up to use "ident" authentication, meaning that it associates Postgres roles with a matching Unix/Linux system account. If a Postgres role exists, it can be signed in by logging into the associated Linux system account.

The installation procedure created a user account called postgres that is associated with the default Postgres role. In order to use Postgres, we'll need to log into that account. You can do that by typing:
```
sudo -i -u postgres
```
You will be asked for your normal user password and then will be given a shell prompt for the postgresuser.

登陆进postgres数据库，创建需要的数据库、用户名，并与之关联
```
stack@phrc-server:~$ sudo -i -u postgres
[sudo] password for stack:
postgres@phrc-server:~$ psql -l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(3 rows)
```
```
postgres@phrc-server:~$ psql -d postgres
psql (9.3.15)
Type "help" for help.

postgres=# \d
No relations found.
postgres=# create user phrcadmin;
CREATE ROLE
postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of
-----------+------------------------------------------------+-----------
 phrcadmin |                                                | {}
 postgres  | Superuser, Create role, Create DB, Replication | {}
```
```
postgres=# create database phrcdb;
CREATE DATABASE

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 phrcdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)

postgres=# alter database phrcdb owner to phrcadmin;
ALTER DATABASE
postgres=# \d
No relations found.
postgres=# \l
                                  List of databases
   Name    |   Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+-----------+----------+-------------+-------------+-----------------------
 phrcdb    | phrcadmin | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres  | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |           |          |             |             | postgres=CTc/postgres
 template1 | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |           |          |             |             | postgres=CTc/postgres
(4 rows)
```
使用以下命令修改用户密码或者创建用户时添加上密码（`create user phrcadmin with password phrcadmin123!`）
```
postgres=#alter role phrcadmin with password phrcadmin123!
```