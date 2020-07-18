---
title: hadoop入门基础
date: 2017-10-17 19:34:05
categories: backEnd
tags: [hadoop]
---
> [Apache Hadoop](http://hadoop.apache.org/)是一个在商业硬件大型集群上运行应用程序的框架。Hadoop框架为应用程序提供透明的可靠性和数据运动。Hadoop实现了一个名为[Map / Reduce](https://wiki.apache.org/hadoop/HadoopMapReduce)的计算范例，其中应用程序分为许多小的工作片段，每个工作片段都可以在集群中的任何节点上执行或重新执行。此外，它还提供了一种在计算节点上存储数据的分布式文件系统（[HDFS](https://wiki.apache.org/hadoop/DFS)），可在集群中提供非常高的聚合带宽。两者的[MapReduce](https://wiki.apache.org/hadoop/MapReduce)和Hadoop分布式文件系统的设计使得节点故障是由框架自动处理。

# hadoop功能与优势
hadoop是什么我们前面已经介绍过了，下面我们主要介绍他的几个核心部分：
1. HDFS：分布式文件系统，存储海量的数据
2. MapReduce：并行处理框架，实现任务分解和调度。
我们一般用hadoop来搭建大型数据仓库、PB级数据的存储、处理、分析、统计等业务。
3. Common: 一组分布式文件系统和通用I/O的组建(序列化、java RPC和持久化数据结构)
4. Pig: 一种数据流语言和运行环境，用以检索非常大的数据集。pig运行在MapRedyce和HdFS集群上。
5. HBase： 一个分布式、按列存储数据库。Hbase使用HDFS作为底层储存，同时支持MapReduce的批量式计算和点查询（随机读取）。
6. ZooKeeper：一个分布式、可用性高的协调服务。ZooKeeper提供分布式锁之类的基本服务，用于构建分布式应用。
7. sqoop： 在数据库和HDFS之间高校传输数据的工具。

hadoop的优势：
1. 高扩展
2. 低成本
3. 成熟的生态圈

# 

# 参考资料
* [在windows平台下安装hadoop](https://www.iwwenbo.com/hadoop-installation-on-windows-without-cygwin/)
* [安装环境JAVA_HOME is incorrectly set问题解决](http://blog.csdn.net/wen3011/article/details/54907731)
* [windows下hadoop本地开发环境的搭建](http://www.voidcn.com/article/p-gdpjxbxw-bz.html)
* [可以会用到的环境搭建资料](http://www.linuxidc.com/Linux/2016-08/134131p2.htm)
* [imooc上的hadoop视屏学习资料](http://www.imooc.com/video/7642)