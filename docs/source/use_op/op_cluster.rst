集群监控
=================

http://master_server代表master服务

集群状态
--------

::

  curl -XGET http://master_server/_cluster/stats


健康状态
--------

::

  curl -XGET http://master_server/_cluster/health
  
查看集群状态及库、表记录数据量(doc_num)。


端口状态
--------

::

  curl -XGET http://master_server/list/server
   

清除锁
--------

::

  curl -XGET http://master_server/clean_lock

在创建表时会对集群加锁，若在此过程中，服务异常，会导致锁不能释放，需要手动清除才能新建表。

副本扩容缩容
--------

::

  curl -XPOST -H "content-type: application/json"  -d'
  {
      "partition_id":1,
      "node_id": 1,
      "method": 0
  }
  ' http://master_server/partition/change_member

method=0: node id 1上添加分片id 1 的副本; method=1: 删除 node id 1 上 分片 id 1 的副本。

集群数据迁移
--------
可以通过下述方式将某个集群的数据拷贝到新集群，实现集群的数据迁移。

1.建立新的目标集群

新集群与待迁移集群节点数保持一致，部署完整的vearch系统，将新集群所有的ps节点的进程kill。

2.元数据同步

通过etcd的镜像功能，将待迁移集群的元数据，拷贝到目标集群。etcdctl是etcd的客户端工具。

操作命令如下：
::

  export ETCDCTL_API=3
  # etcd 镜像的原理是读取一行一行的key，写入到另外的集群中。其中：sourceMasterIP是原始集群master的一个节点，targetMasterIP是目标集群master的一个节点。
  # ETCDCTL_API=3 ./etcdctl make-mirror target --endpoints=source
  ETCDCTL_API=3 ./etcdctl make-mirror ${targetMasterIP}:2370 --endpoints=${sourceMasterIP}:2370


3.删除目标集群的 /server 元信息

可以etcdctl get 先查询server 元信息的前缀，不同的版本可能包含不同的前缀信息
::

  export ETCDCTL_API=3
  ./etcdctl --endpoints=http://${targetMasterIP}:2370 del /server --prefix


4.拷贝向量数据
::

  scp -r root@sourcePsIP:/export/vdb/baud root@targetPsIP:/export/vdb
  ... 省略其他ip的数据拷贝

sourcePsIP是待迁移集群PS节点的ip，targetPsIP是目标集群ps节点的ip。此处只需要保证待迁移集群与目标集群的ps节点ip一对一进行迁移即可，不需要特殊顺序。
