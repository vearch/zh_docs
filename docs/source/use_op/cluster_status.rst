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
   

