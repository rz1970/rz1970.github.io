---
title: Upgrade GP from 5.x to 5.x
tags: greenplum
---

```
# su - gpadmin
$ gpstop -a
# chown -R gpadmin:gpadmin /usr/local/greenplum*
$ rm /usr/local/greenplum-db
$ ln -s /usr/local/greenplum-db-5.2.0 /usr/local/greenplum-db
$ source ~/.bashrc
$ gpseginstall -f hostfile
# su - gpadmin
$ gpstart
```