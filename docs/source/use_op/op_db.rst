库操作
=================

查看库列表
--------


::

   curl -XGET http://xxxxxx/list/db
 

创建库
--------

::

   curl -XPUT -H "content-type:application/json" -d '{
     "name": "db_name"
   }' http://xxxxxx/db/_create


查看库
--------

::

   curl -XGET http://xxxxxx/db/$db_name


删除库
--------

::

   curl -XDELETE http://xxxxxx/db/$db_name


查看库表空间
--------

::

   curl -XGET http://xxxxxx/list/space?db=$db_name



