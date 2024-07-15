鉴权
=================

http://${VEARCH_URL}代表vearch服务

RBAC 鉴权模式
---------------

分为用户（user）、role (角色)、权限三部分。权限定义对应接口资源的访问权限，角色则定义包含哪些权限，用户则归属于某种角色。

权限包含三种权限：WriteOnly、ReadOnly、WriteRead.

接口资源：

+-------------------+--------------------------+---------------------------------+
|   接口资源类型    |         字段含义         |              备注               |
+===================+==========================+=================================+
| ResourceAll       | 所有接口资源             | root角色的权限                  |
+-------------------+--------------------------+---------------------------------+
| ResourceCluster   | 集群接口                 | /cluster 前缀的接口             |
+-------------------+--------------------------+---------------------------------+
| ResourceServer    | partition server接口资源 | /servers 前缀的接口             |
+-------------------+--------------------------+---------------------------------+
| ResourcePartition | 自定义分片类型           | /partitions 前缀的接口          |
+-------------------+--------------------------+---------------------------------+
| ResourceDB        | 自定义分片字段           | /dbs 前缀的接口                 |
+-------------------+--------------------------+---------------------------------+
| ResourceSpace     | 自定义分片范围           | /dbs/$db_name/spaces 前缀的接口 |
+-------------------+--------------------------+---------------------------------+
| ResourceDocument  | 自定义分片类型           | /document 前缀的接口            |
+-------------------+--------------------------+---------------------------------+
| ResourceIndex     | 自定义分片字段           | /index 前缀的接口               |
+-------------------+--------------------------+---------------------------------+
| ResourceAlias     | 自定义分片范围           | /alias 前缀的接口               |
+-------------------+--------------------------+---------------------------------+
| ResourceUser      | 自定义分片类型           | /uesrs 前缀的接口               |
+-------------------+--------------------------+---------------------------------+
| ResourceRole      | 自定义分片字段           | /roles 前缀的接口               |
+-------------------+--------------------------+---------------------------------+
| ResourceConfig    | 自定义分片范围           | /config 前缀的接口              |
+-------------------+--------------------------+---------------------------------+
| ResourceCache     | 自定义分片范围           | /cache 前缀的接口               |
+-------------------+--------------------------+---------------------------------+

集群默认已经创建root用户和 root角色。

创建角色
>>>>>>>>>>>>

::
   
   curl -XPOST -H "content-type: application/json" -d'
    {
        "name": "test",
        "privileges": {
            "ResourceDocument": "ReadOnly"
        }
    }
   ' http://${VEARCH_URL}/roles

删除角色
>>>>>>>>>>>>

::

  curl -XDELETE http://${VEARCH_URL}/roles/$role_name

查看角色
>>>>>>>>>>>>

::

  curl -XGET http://${VEARCH_URL}/roles/$role_name


修改角色权限
>>>>>>>>>>>>>>>>

添加权限

::
   
   curl -XPUT -H "content-type: application/json" -d'
    {
        "name": "test",
        "operator": "Grant",
        "privileges": {
            "ResourceSpace": "WriteRead"
        }
    }
   ' http://${VEARCH_URL}/roles

删除权限

::
   
   curl -XPUT -H "content-type: application/json" -d'
    {
        "name": "test",
        "operator": "Revoke",
        "privileges": {
            "ResourceDocument": "ReadOnly"
        }
    }
   ' http://${VEARCH_URL}/roles

创建用户
>>>>>>>>>>>>>>>>

::
   
   curl -XPOST -H "content-type: application/json" -d'
    {
        "name": "test",
        "password": "******",
        "role_name": "test"
    }
   ' http://${VEARCH_URL}/users

删除用户
>>>>>>>>>>>>>>>>

::

  curl -XDELETE http://${VEARCH_URL}/users/$user_name

查看用户
--------

::

  curl -XGET http://${VEARCH_URL}/users/$user_name


修改用户
>>>>>>>>>>>>>>>>

修改用户角色

::
   
   curl -XPUT -H "content-type: application/json" -d'
    {
        "name": "test",
        "role_name": "test2"
    }
   ' http://${VEARCH_URL}/users

修改用户密码

::
   
   curl -XPUT -H "content-type: application/json" -d'
    {
        "name": "test",
        "password": "******",
        "old_password": "******"
    }
   ' http://${VEARCH_URL}/users