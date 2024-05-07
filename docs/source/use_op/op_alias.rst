别名操作
=================

别名可以是一个简短的字符串，方便标识和访问对应的表空间。通常别名会替换 Space 的名称以方便业务切换等场景。

http://master_server代表master服务，$db_name是创建的库名, $space_name是创建的表空间名

创建表空间别名
----------------
::
 
  curl -XPOST http://master_server/alias/$alias_name/dbs/$db_name/spaces/$space_name


更新表空间别名
----------------
::
 
  curl -XPUT http://master_server/alias/$alias_name/dbs/$db_name/spaces/$space_name


获取别名详情
----------------
::
 
  curl -XGET http://master_server/alias/$alias_name

删除表空间别名
----------------
::
 
  curl -XDELETE http://master_server/alias/$alias_name