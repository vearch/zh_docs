概述
========

Vearch 是对大规模深度学习向量进行高效相似搜索的弹性分布式系统。

整体架构
-----------------------

.. image:: pic/vearch-arch.png
   :align: center
   :scale: 50 %
   :alt: Architecture

数据模型: 空间，文档，向量，标量。

组件: Master，Routerm，PartitionServer。

Master: 负责schema管理，集群级别的源数据和资源协调。

Router: 提供RESTful API: create、delete、search、update; 请求路由转发及结果合并。

PartitionServer(PS): 基于raft复制的文档分片; Gamma是向量搜索引擎,它提供了存储、索引和检索向量、标量的能力。


功能简介
-----------------------

1、单个文档单向量。

2、单个文档单个数据源多个向量。

3、单个文档多个数据源多个向量。

4、数值字段过滤。

5、支持添加、搜索的批量操作。


系统特性
-----------------------
1、C++实现的gamma引擎，保证向量的快速检索

2、支持内积与L2方法计算向量距离

3、支持内存与磁盘两种数据存储方式，支持超大数据规模

4、基于raft协议实现数据多副本存储

