库操作
=================

http://${VEARCH_URL}代表vearch服务, $db_name是创建的库名

查看集群中所有库
--------

::

  curl -XGET http://${VEARCH_URL}/dbs
 

创建库
--------

::

  curl -XPOST http://${VEARCH_URL}/dbs/$db_name


查看指定库
--------

::

  curl -XGET http://${VEARCH_URL}/dbs/$db_name


删除库
--------

::

  curl -XDELETE http://${VEARCH_URL}/dbs/$db_name

若库下存在表空间则无法删除


查看指定库下所有表空间
------------------------

::

  curl -XGET http://${VEARCH_URL}/dbs/$db_name/spaces
