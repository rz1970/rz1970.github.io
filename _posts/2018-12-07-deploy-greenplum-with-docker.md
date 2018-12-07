---
title: 在docker中部署greenplum
tags: [greenplum,docker]
---

# Prepare

1. Docker version 18.09.0, build 4d60db4
2. docker-compose version 1.23.2, build 1110ad01
3. centos:7.5.1804 的 docker 镜像
4. greenplum-db-5.14.0-rhel7-x86_64.bin 二进制文件

# 准备 docker 环境

1.基于 centos 制作 greenplum 需要的 docker 镜像。Dockerfile 如下：

```Dockerfile
FROM centos:7.5.1804
RUN yum -y update; yum clean all
RUN yum install -y \
    net-tools \
    ntp \
    openssh-server \
    openssh-clients \
    less \
    iproute \
    which; yum clean all
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N ''
RUN groupadd gpadmin
RUN useradd gpadmin -g gpadmin
RUN echo gpadmin | passwd gpadmin --stdin
ENTRYPOINT ["/usr/sbin/sshd", "-D"]
```

执行 `docker build -t centos-ssh .` 创建 docker 镜像

2.通过 docker-compose 起来所需要的 greenplum 集群环境。docker-compose.yaml 文件内容如下：

```YAML
version: "3.2"

services:

  mdw:
    image: centos-ssh
    container_name: gpdb-mdw
    tty: true

  sdw1:
    image: centos-ssh
    container_name: gpdb-sdw1
    tty: true

  sdw2:
    image: centos-ssh
    container_name: gpdb-sdw2
    tty: true
```

执行 `docker-compose up -d` 运行所需要的环境

# 部署 Greenplum

1.通过 `docker cp` 命令将 greenplum-db-5.14.0-rhel7-x86_64.bin 拷贝到 mdw 的容器中。

```bash
docker cp greenplum-db-5.14.0-rhel7-x86_64.bin gpdb-mdw:/home/gpadmin
```

2.进入 gpdb-mdw 容器中，并切换到gpadmin用户下，查看刚拷贝的bin文件。

```bash
docker exec -it gpdb-mdw /bin/bash
[root@67758e384c8c /]# su - gpadmin
Last login: Fri Dec  7 07:58:14 UTC 2018 on pts/1
[gpadmin@67758e384c8c ~]$ ls
greenplum-db-5.14.0-rhel7-x86_64.bin
```

3.执行greenplum的bin文件，将其安装在`/home/gpadmin`目录下.

```bash
[root@mdw ~]# bash greenplum-db-5.14.0-rhel7-x86_64.bin
```

参考如下：

```bash
********************************************************************************
Do you accept the Pivotal Database license agreement? [yes|no]
********************************************************************************

yes

********************************************************************************
Provide the installation path for Greenplum Database or press ENTER to
accept the default installation path: /usr/local/greenplum-db-5.14.0
********************************************************************************

/home/gpadmin/greenplum-db-5.14.0

********************************************************************************
Install Greenplum Database into /home/gpadmin/greenplum-db-5.14.0? [yes|no]
********************************************************************************

yes

********************************************************************************
/home/gpadmin/greenplum-db-5.14.0 does not exist.
Create /home/gpadmin/greenplum-db-5.14.0 ? [yes|no]
(Selecting no will exit the installer)
********************************************************************************

yes

Extracting product to /home/gpadmin/greenplum-db-5.14.0

********************************************************************************
Installation complete.
Greenplum Database is installed in /home/gpadmin/greenplum-db-5.14.0

Pivotal Greenplum documentation is available
for download at http://gpdb.docs.pivotal.io
********************************************************************************
[gpadmin@67758e384c8c ~]$ ls
greenplum-db  greenplum-db-5.14.0  greenplum-db-5.14.0-rhel7-x86_64.bin
```

4.Create a file called hostlist with all hosts as following and seglist without mdw in `/home/gpadmin/gpconfig/` dir.

```hostlist.text
mdw
sdw1
sdw2
```

```seglist.text
sdw1
sdw2
```

5.Source the path file from your master host's Greenplum Database installation directory

```bash
[gpadmin@67758e384c8c ~]$ source ~/greenplum-db/greenplum_path.sh
```

6.Config ssh key exchange(master server and gpadmin user)

```bash
[gpadmin@67758e384c8c ~]$ gpssh-exkeys -f /home/gpadmin/gpconfig/hostlist
```

7.Run the `gpseginstall` utility referencing the hostfile_exkeys file you just created

```bash
[gpadmin@67758e384c8c ~]$ gpseginstall -f /home/gpadmin/gpconfig/seglist
```

8.Confirm installation

```bash
[gpadmin@67758e384c8c ~]$ gpssh -f ~/gpconfig/hostlist -e ls -l $GPHOME
```

9.Creating the Data Storage Areas

on master

```bash
[gpadmin@67758e384c8c ~]$ mkdir -p ~/data/master
```

on segments

```bash
[gpadmin@67758e384c8c ~]$ gpssh -f ~/gpconfig/seglist -e "mkdir -p ~/data/primary"
[sdw2] mkdir -p ~/data/primary
[sdw1] mkdir -p ~/data/primary
[gpadmin@67758e384c8c ~]$ gpssh -f ~/gpconfig/seglist -e "mkdir -p ~/data/mirror"
[sdw2] mkdir -p ~/data/mirror
[sdw1] mkdir -p ~/data/mirror
```

10.Make a copy of the gpinitsystem_config file to use as a starting point and edit it as you need

```bash
[gpadmin@67758e384c8c ~]$ cp /home/gpadmin/greenplum-db/docs/cli_help/gpconfigs/gpinitsystem_config ~/gpconfig/gpinitsystem_config
```

11.modify two line as following

```plain
declare -a DATA_DIRECTORY=(/home/gpadmin/data/primary)
MASTER_DIRECTORY=/home/gpadmin/data/master
```

12.run the initialization utility

```bash
[gpadmin@mdw ~]$ gpinitsystem -c gpconfig/gpinitsystem_config -h gpconfig/seglist
```

13.see the `Greenplum Database instance successfully created.` You install successfully