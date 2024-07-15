鉴权
=================

http://${VEARCH_URL}代表vearch服务

RBAC 鉴权模式
---------------

分为用户（user）、role (角色)、权限三部分。权限定义对应接口资源的访问权限，角色则定义包含哪些权限，用户则归属于某种角色。

权限包含三种权限：WriteOnly、ReadOnly、WriteRead.

接口资源：
    ResourceAll
	ResourceCluster
	ResourceServer
	ResourcePartition
	ResourceDB
	ResourceSpace
	ResourceDocument
	ResourceIndex
	ResourceAlias
	ResourceUser
	ResourceRole
	ResourceConfig
	ResourceCache

集群默认已经创建root用户和 root角色。

创建角色
----------

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
----------

::

  curl -XDELETE http://${VEARCH_URL}/roles/$role_name

查看角色
----------

::

  curl -XGET http://${VEARCH_URL}/roles/$role_name


修改角色权限
--------------

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
--------

::
   
   curl -XPOST -H "content-type: application/json" -d'
    {
        "name": "test",
        "password": "******",
        "role_name": "test"
    }
   ' http://${VEARCH_URL}/users

删除用户
--------

::

  curl -XDELETE http://${VEARCH_URL}/users/$user_name

查看用户
--------

::

  curl -XGET http://${VEARCH_URL}/users/$user_name


修改用户
--------------

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