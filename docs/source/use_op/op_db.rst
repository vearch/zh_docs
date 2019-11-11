库操作
=================

http://master_server代表master服务，$db_name是创建的库名

查看集群中所有库
--------

::

  curl -XGET http://master_server/list/db
 

创建库
--------

::

  curl -XPUT -H "content-type:application/json" -d '{
      "name": "db_name"
  }
  ' http://master_server/db/_create


查看库
--------

::

  curl -XGET http://master_server/db/$db_name


删除库
--------

::

  curl -XDELETE http://master_server/db/$db_name

若库下存在表空间则无法删除


查看指定库下所有表空间
--------

::

  curl -XGET http://master_server/list/space?db=$db_name


