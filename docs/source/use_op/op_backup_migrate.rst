备份与迁移
=================
http://${VEARCH_URL} 代表vearch服务, DB_NAME是创建的库名, SPACE_NAME是创建的空间名

vearch可以将集群表空间备份远程存储, 以便于迁移或者恢复, 在使用备份功能前, 需要先配置S3存储


备份
----------------

::

  curl --location 'http://${VEARCH_URL}/backup/dbs/${DB_NAME}/spaces/${SPACE_NAME}' \
  --data '{
      "command": "create",
      "with_schema": true,
      "s3_param": {
          "access_key": "USER_ACCESS_KEY",
          "bucket_name": "USER_BUCKET_NAME",
          "endpoint": "S3_ENDPOINT",
          "secret_key": "USER_SECRET",
          "use_ssl": true
      }
  }'

with_schema: 设置是否备份表信息, 默认为true, 如果设置为true, 则备份表信息, 否则只备份数据

access_key: S3存储的access key

bucket_name: S3存储的bucket名称

endpoint: S3存储的endpoint

secret_key: S3存储的secret key

use_ssl: 是否使用ssl

恢复
----------------

::

  curl --location 'http://${VEARCH_URL}/backup/dbs/${DB_NAME}/spaces/${SPACE_NAME}' \
  --data '{
      "command": "restore",
      "with_schema": true,
      "s3_param": {
          "access_key": "USER_ACCESS_KEY",
          "bucket_name": "USER_BUCKET_NAME",
          "endpoint": "S3_ENDPOINT",
          "secret_key": "USER_SECRET",
          "use_ssl": true
      }
  }'

副本扩缩和分片迁移
----------------

::

  curl -XPOST -H "content-type: application/json"  -d'
  {
      "partition_ids": [1],
      "node_id": 1,
      "method": 0
  }
  ' http://${VEARCH_URL}/partitions/change_member

method=0: node id 1上添加分片id 1 的副本; method=1: 删除 node id 1 上 分片 id 1 的副本。

通过在新节点上新增分片的副本然后删除之前的分片实现分片迁移


集群数据迁移
--------
可以通过下述方式将某个集群的数据拷贝到新集群, 实现集群的数据迁移。

1.建立新的目标集群

新集群与待迁移集群节点数保持一致, 部署完整的vearch系统, 将新集群所有的ps节点的进程kill。

2.元数据同步

通过etcd的镜像功能, 将待迁移集群的元数据, 拷贝到目标集群。etcdctl是etcd的客户端工具。

操作命令如下：
::

  export ETCDCTL_API=3
  # etcd 镜像的原理是读取一行一行的key, 写入到另外的集群中。其中: sourceMasterIP是原始集群master的一个节点, targetMasterIP是目标集群master的一个节点。
  # ETCDCTL_API=3 ./etcdctl make-mirror target --endpoints=source
  ETCDCTL_API=3 ./etcdctl make-mirror ${targetMasterIP}:2370 --endpoints=${sourceMasterIP}:2370


3.删除目标集群的 /server 元信息

可以etcdctl get 先查询server 元信息的前缀, 不同的版本可能包含不同的前缀信息
::

  export ETCDCTL_API=3
  ./etcdctl --endpoints=http://${targetMasterIP}:2370 del /server --prefix


4.拷贝向量数据
::

  scp -r root@sourcePsIP:/export/vdb root@targetPsIP:/export/vdb
  ... 省略其他ip的数据拷贝

sourcePsIP是待迁移集群PS节点的ip, targetPsIP是目标集群ps节点的ip。此处只需要保证待迁移集群与目标集群的ps节点ip一对一进行迁移即可, 不需要特殊顺序。