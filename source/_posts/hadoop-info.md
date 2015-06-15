title: hadoop 2.x 基础
date: 2015-06-14 14:33:04
categories: Study Notes
tags: [Hadoop]
---
# HDFS 模块功能
- NameNode为主节点，**存储文件的元数据**如文件名，文件目录结构，文件属性（生成时间，副本数，文件权限），以及每个文件的块列表和块所在的DataNode等。
- DataNode在本地文件系统**存储文件块数据，以及块数据的校验和**。
- Secondary NameNode用来**监控HDFS状态的辅助后台程序**，每隔一段时间**获取HDFS元数据的快照**。


# YARN（云操作系统） 模块功能
ResourceManager

> - 处理客户端请求
> - 启动/监控ApplicationMaster
> - 监控NodeManager
> - 资源分配与调度

NodeManager

> - 单个节点上的资源管理
> - 处理来自ResourceManager的命令
> - 处理来自ApplicationMaster的命令

ApplicationMaster

> - 数据切分
> - 为应用程序申请资源，并分配给内部任务
> - 任务监控与容错

Container

> -对任务进行环境的抽象，封装了cpu、内存等多维资源以及环境变量、启动命令等任务运行相关的信息

# 离线计算框架MapReduce

将计算过程分为两个阶段，Map和Reduce

> - Map阶段并行处理输入数据
> - Reduce阶段对Map结果进行汇总

Shuffle连接Map和Reduce两个阶段

> - Map Task将数据写到本地磁盘
> - Reduce Task从每个Map Task上读取一份数据

仅适合离线批处理

> - 具有很好的容错性和扩展性
> - 适合简单的批处理任务

缺点明显

> - 启动开销大、过多使用磁盘导致效率低下等
