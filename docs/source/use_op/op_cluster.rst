集群监控
=================

http://${VEARCH_URL}代表vearch服务

集群状态
--------

::

  curl -XGET http://${VEARCH_URL}/cluster/stats


健康状态
--------

::

  curl -XGET http://${VEARCH_URL}/cluster/health
  
查看集群状态及库


::

  curl -XGET http://${VEARCH_URL}/cluster/health?db=$db_name&space=$space_name&detail=true

查看集群状态并且指定库和表空间

server状态
--------

::

  curl -XGET http://${VEARCH_URL}/servers

partition状态
----------------

::

  curl -XGET http://${VEARCH_URL}/partitions

清除锁
--------

::

  curl -XGET http://${VEARCH_URL}/clean_lock

在创建表时会对集群加锁, 若在此过程中, 服务异常, 会导致锁不能释放, 需要手动清除才能新建表。
