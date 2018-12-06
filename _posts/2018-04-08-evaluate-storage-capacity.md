---
title: Greenplum 评估存储容量
tags: greenplum
---

## 计算磁盘的可用容量

1. 首先计算一台segment节点的磁盘的可用存储容量
+ 获取磁盘的原始容量
```
raw_capacity = disk_size * number_of_disks
```

+ 除去文件系统格式所占用的开销(大约10%) 和 Raid所使用的容量。在此假如使用`RAID-10`
```
formatted_disk_space = (raw_capacity * 0.9) / 2
```
+ 为了性能最有，不能完全占用磁盘空间。大概至`70%`为宜
```
usable_disk_space = formatted_disk_space * 0.7
```
所以：粗略的估计，一个segment节点，采用`RAID-10`的方式做磁盘阵列，则其可用的磁盘空间大概为原始容量的`31.5%`

2. 以上获取了一个segment节点最大的可用存储容量，那么GPDB上用户的可以使用的用户数据量为：
```
With mirrors:     usable_disk_space = (2 * U) + U/3

Without mirrors:  usable_disk_space = U + U/3
```
Note: **U** is short for UserData. **U/3** is some space be reserved as a working area for active queries，高并发情况下，这个值可能需要更大些。粗略估算下，做了mirror的，数据库可以使用的空间大约是原始空间的`13.5%`，不做mirror的，数据库使用的空间大约是原始空间的`23.6%`

## 计算用户数据的大小

1. 首先，和所有的数据库系统一样，将原始数据加载进入数据库后，其大小会变为原来的1.4倍左右。当然，可以根据 the data types you are using, table storage type, in-database compression 等等来改变其大小
+ Page Overhead: 数据加载进入数据库后，被分割成每页32KB大小的Page。每个page消耗20 bytes
+ Row Overhead: regular 'heap' storage table：每个row 是24 bytes，对于'append-optimized'的，则每个row 4 bytes
+ Attribute Overhead: depend on the data type chosen
+ Indexes: distributed across the segment hosts as is table data. Default is B-tree. Index size depends on the number of unique values in the index and the data to be inserted
```
B-tree: unique_values * (data_type_size + 24 bytes)

Bitmap: (unique_values * number_of_rows * 1 bit * compression_ratio / 8) + (unique_values * 32)
```

## 计算元数据和日志所需要的空间

+ System Metadata：每个segment实例或master实例大约需要 20 MB for the system catalogs and metadata
+ Write Ahead Log(WAL)：每个segment实例或者master实例都有WAL文件。每个WAL文件的大小为：64M。WAL文件在一个GPDB集群中的数量为：
 ```
 2 * checkpoint_segments + 1
 ```
 一个GPDB集群的默认的checkpoint_segments为8个实例。这意味着需要给每个节点上的实例分配1088M大小的WAL空间。
 + Greenplum Database Log Files： 每个实例都会产生很多日志，需要经常归档，使得不至于太大。
 + Command Center Data：gpperfmon 监控的数据库，其大小取决于要保留多久的历史记录