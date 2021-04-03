---
title: HDFS的优缺点及MapReduce的运行机制
date: 2021-03-21 13:15:39
tags: 大数据
categories: 大数据
---

### HDFS的优缺点及MapReduce的运行机制

###### HDFS

```
优点

1、高容错性自动保存多个副本，丢失后可自动恢复。

2、适合处理大数据

3、可构建廉价服务器，提高性能。

缺点:

1、不适合低延时访问

2、无法高效的对大量小文件存储

3、不支持文件并发访问，文件随机修改。
```

NameNode : 就是Master,它是一个主管、管理者

```
1、管理HDFS的名称空间
2、配置副本策略
3、管理数据块的映射信息
4、处理客户端读写请求
```

DataNode:就是Slave。NameNode下达命令，DataNode执行实际的操作。

```
1、存储实际的数据块
2、执行数据块的读写操作
```

Client：就是客户端

```
1、文件切分，将文件切分为一个个Block(默认128M)，然后上传。
2、与NameNode交互，获取文件的位置信息
3、与DataNode交互，读取或者写入数据
4、Client提供一些命令来管理HDFS，比如NameNode格式化。
5、Client可通过一些命令来访问HDFS,比如对HDFS增删改查操作。
```

Secondary NameNode :当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。

```
1、辅助NameNode，分担工作量，日入定期合并Fsimage和Edits，并推送给NameNode;
2、在紧急情况下，可恢复NameNode。
```

HDFS块的大小设置主要取决于磁盘传输速率，寻址时间为传输时间的1%时，则为最佳状态。

安全模式：

上传文件时一定是非安全模式。集群启动后，自动退出安全模式。

```t
a、hdfs dfsadmin -safemode get 查看安全模式的状态。
b、hdfs dfsadmin -safemode enter/leave 进入/离开安全模式
c、hdfs dfsadmin -safemode wait等待安全模式
```

解决存储小文件的办法之一：

HDFS存档文件或者是HAR文件，是一个高效的文件存档工具，它将文件存入HDFS块，在减少NameNode内存的同时，允许对文件的透明访问。具体来说，HDFS存档文件对内还是一个个 独立文件,对NameNode还是一个整体，减少了NameNode的内存 。

快照管理：

快照相当于对文件做一个备份，并不会立即复制所有文件，而是指向同一个文件，当写入发生时，才会产生新文件。

```
1、hdfs dfadmin -allowSnapshot 路径 开启快照
2、hdfs dfadmin -disallowSnapshot 路径 关闭快照
3、hdfs dfs -createSnapshot 路径  创建快照
4、hdfs lsSnapshottableDir  查看快照
5、hdfs snapshotDiff 路径1  .  路径2  比较两个路径的不同
```



##### MapReduce:

分布式运算程序编程框架，是用户开发“基于Hadoop”的编程框架。

```
优势：
1、MapReduce易于编程
2、良好的可扩展性。
3、高容错性。
4、适合PB级以上海量数据的离线存储。
缺点：
1、不擅长实时计算
2、不擅长流式计算
3、不擅长DAG(有向图)的计算。
```

核心思想：

```
一个完整的MapReduc 程序在分布式运行时有三类实例进程。

MrAppMaster:负责整个程序的过程调度及状态协调

MapTask:负责Map阶段的整个数据流程处理。

ReduceTask:负责reduce阶段的整个数据流程处理。
```

Hadoop序列化:

```
java的序列化是一个重量级的序列化框架，一个对象被序列化后，会附带很多的额外信息，不便于在网络中高效的传输。所以，Hadoop自己开发了一套序列化机制。
  特点：a:紧凑（高效使用存储空间）
        b:快速 (读写数据的额外开销小)
        c:可扩展（随着通信协议的升级而升级）
        d:互操作（支持多语言的交互）
```

数据切片与MapTask并行度决定机制:

```
1、一个job 的Map阶段并行度由客户端提交job时的切片数决定。

2、每一个Split切片分配一个 并行处理实例。

3、默认情况下，切片大小=BlockSize

4、切片时不考虑数据集整体，而是逐个对每一个文件切片。
```

