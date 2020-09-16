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

method=0 add partition:1 to node:1; method=1 delete partition:1 from node:1
