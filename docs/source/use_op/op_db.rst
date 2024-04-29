库操作
=================

http://master_server代表master服务，$db_name是创建的库名

查看集群中所有库
--------

::

  curl -XGET http://master_server/dbs
 

创建库
--------

::

  curl -XPOST http://master_server/dbs/$db_name


查看指定库
--------

::

  curl -XGET http://master_server/dbs/$db_name


删除库
--------

::

  curl -XDELETE http://master_server/dbs/$db_name

若库下存在表空间则无法删除


查看指定库下所有表空间
--------

::

  curl -XGET http://master_server/dbs/$db_name/spaces


