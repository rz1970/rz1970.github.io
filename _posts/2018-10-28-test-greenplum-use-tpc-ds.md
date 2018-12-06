---
title: 使用 TPC-DS 生成测试数据和查询语句
tags: greenplum
---

1 从其官方网站下载 TPC-DS 源码。[TPC - Current Specifications](http://www.tpc.org/tpc_documents_current_versions/current_specifications.asp)

2 在centos 7.5环境下编译。
```
# yum -y install gcc gcc-c++ expect uzip
# unzip v2.8.0rc4.zip
# cd v2.8.0rc4/tools
# make
```
3 生成测试数据, 创建data目录，放置生成的测试数据
```
# mkdir /root/data
# cd v2.8.0rc4/tools
# ./dsdgen -SCALE 1GB -DIR /root/data
```
4 并行生成测试数据
```
# ./dsdgen -SCALE 1GB -DIR /root/data -parallel 4 -child 4
```
5 生成查询语句
```
# ./dsqgen -input ../query_templates/templates.lst -directory ../query_templates -dialect oracle -scale 1GB -OUTPUT_DIR /root/data/query_oracle
```