表空间操作
=================

http://${VEARCH_URL}代表master服务, $db_name是创建的库名, $space_name是创建的表空间名

创建表空间
-----------

::
   
   curl -XPOST -H "content-type: application/json" -d'
    {
        "name": "space1",
        "partition_num": 1,
        "replica_num": 3,
        "fields": [
            {
                "name": "field_string",
                "type": "string"
            },
            {
                "name": "field_int",
                "type": "integer"
            },
            {
                "name": "field_float",
                "type": "float",
                "index": {
                    "name": "field_float",
                    "type": "SCALAR"
                }
            },
            {
                "name": "field_string_array",
                "type": "stringArray",
                "index": {
                    "name": "field_string_array",
                    "type": "SCALAR"
                }
            },
            {
                "name": "field_int_index",
                "type": "integer",
                "index": {
                    "name": "field_int_index",
                    "type": "SCALAR"
                }
            },
            {
                "name": "field_vector",
                "type": "vector",
                "dimension": 128,
                "index": {
                    "name": "gamma",
                    "type": "IVFPQ",
                    "params": {
                        "metric_type": "InnerProduct",
                        "ncentroids": 2048,
                        "nlinks": 32,
                        "efConstruction": 40
                    }
                }
            }
        ]
    }
   ' http://${VEARCH_URL}/dbs/$db_name/spaces


参数说明:

+---------------+----------+--------+----------+------------------------------------------------+
|   字段标识    | 字段含义 |  类型  | 是否必填 |                      备注                      |
+===============+==========+========+==========+================================================+
| name          | 空间名称 | string | 是       |                                                |
+---------------+----------+--------+----------+------------------------------------------------+
| partition_num | 分片数量 | int    | 是       | 单节点数据承载有效, 多个分片提升集群数据承载量 |
+---------------+----------+--------+----------+------------------------------------------------+
| replica_num   | 副本数量 | int    | 是       | 通常设置为3来实现高可用                        |
+---------------+----------+--------+----------+------------------------------------------------+
| fields        | 空间配置 | json   | 是       | 定义表字段及类型,是否创建索引                  |
+---------------+----------+--------+----------+------------------------------------------------+

1、name 不能为空, 不能以数字或下划线开头, 不要包含-（中划线），尽量不使用特殊字符等。

2、partition_num 指定表空间数据分片数量, 不同的分片可分布在不同的机器, 来避免单台机器的资源限制。

3、replica_num 副本数量, 建议设置成3, 表示每个分片数据有两个备份, 保证数据高可用。

index配置:

+----------+--------------+--------+----------+------+
| 字段标识 |   字段含义   |  类型  | 是否必填 | 备注 |
+==========+==============+========+==========+======+
| name     | 索引名称     | string | 是       |      |
+----------+--------------+--------+----------+------+
| type     | 索引类型     | string | 是       |      |
+----------+--------------+--------+----------+------+
| params   | 索引参数配置 | json   | 否       |      |
+----------+--------------+--------+----------+------+

1、index type 索引类型, 目前支持二大类共七种类型, 标量索引: SCALAR; 向量索引: IVFPQ, HNSW, GPU, IVFFLAT, BINARYIVF, FLAT, 详细可看链接
https://github.com/vearch/vearch/wiki/Vearch%E7%B4%A2%E5%BC%95%E4%BB%8B%E7%BB%8D%E5%92%8C%E5%8F%82%E6%95%B0%E9%80%89%E6%8B%A9 。

标量索引只需设置name和type即可。

不同的向量索引类型需要的参数配置及默认值如下:

IVFPQ:

IVFPQ可以与HNSW和OPQ组合使用。 如果要使用HNSW, 建议将ncentroids设置为较大的值。而在组合使用OPQ时, 
训练占用的内存为 2 * training_threshold * dimension * sizeof(float), 因此对于HNSW和OPQ的组合使用, 
训练将占用更多的内存并花费较长时间, 故要特别注意training_threshold的设置, 防止使用的太多内存。

training_threshold: 对于IVFPQ, 在建立索引之前需要训练, 因此需要将training_threshold设置为合适的值,
training_threshold可以是 ncentroids * 39 到 ncentroids * 256 之间的值。

如何组合使用HNSW和OPQ由params控制。如果同时设置HNSW和OPQ, 则将使用OPQ + IVF + HNSW + PQ, 
建议将OPQ的nsubvector设置为与PQ的nsubvector相同。如果只想使用IVF + HNSW + PQ, 
则只需要设置HNSW。如果您只想使用IVFPQ, 则无需在params中设置HNSW或OPQ。

+--------------------+---------------------------+--------+----------+---------------------------------------------------------------------+
|      字段标识      |         字段含义          |  类型  | 是否必填 |                                备注                                 |
+====================+===========================+========+==========+=====================================================================+
| metric_type        | 计算方式                  | string | 是       | L2或者InnerProduct                                                  |
+--------------------+---------------------------+--------+----------+---------------------------------------------------------------------+
| ncentroids         | 聚类中心数量              | int    | 是       | 默认2048                                                            |
+--------------------+---------------------------+--------+----------+---------------------------------------------------------------------+
| nsubvector         | PQ拆分子向量大小          | int    | 否       | 默认为向量维度除以2                                                 |
+--------------------+---------------------------+--------+----------+---------------------------------------------------------------------+
| bucket_init_size   | 倒排链表(IVF)初始化的大小 | int    | 否       | 默认1000                                                            |
+--------------------+---------------------------+--------+----------+---------------------------------------------------------------------+
| bucket_max_size    | 倒排链表(IVF)最大容量     | int    | 否       | 默认1280000                                                         |
+--------------------+---------------------------+--------+----------+---------------------------------------------------------------------+
| training_threshold | 训练的数据量              | int    | 否       | 默认ncentroids * 39,是每个分片训练需要的数据量, 不是space表的数据量 |
+--------------------+---------------------------+--------+----------+---------------------------------------------------------------------+
| nprobe             | 检索时查找的聚类中心数量  | int    | 否       | 默认80                                                              |
+--------------------+---------------------------+--------+----------+---------------------------------------------------------------------+

::
 
  "type": "IVFPQ",
  "params": {
      "metric_type": "InnerProduct",
      "ncentroids": 2048,
      "nsubvector": 64
  }

您可以这样设置hnsw或opq: 

::

  "type": "IVFPQ",
  "params": {
      "metric_type": "InnerProduct",
      "ncentroids": 65536,
      "nsubvector": 64,
      "hnsw" : {
          "nlinks": 32,
          "efConstruction": 200,
          "efSearch": 64
      }
  }

HNSW:

+----------------+------------------------------------------+--------+----------+--------------------+
|    字段标识    |                 字段含义                 |  类型  | 是否必填 |        备注        |
+================+==========================================+========+==========+====================+
| metric_type    | 计算方式                                 | string | 是       | L2或者InnerProduct |
+----------------+------------------------------------------+--------+----------+--------------------+
| nlinks         | 节点邻居数量                             | int    | 是       | 默认32             |
+----------------+------------------------------------------+--------+----------+--------------------+
| efConstruction | 构图时寻找节点邻居过程中在图中遍历的深度 | int    | 是       | 默认40             |
+----------------+------------------------------------------+--------+----------+--------------------+
| efSearch       | 检索时寻找节点邻居过程中在图中遍历的深度 | int    | 否       | 默认40             |
+----------------+------------------------------------------+--------+----------+--------------------+

::

  "type": "HNSW",
  "params": {
      "metric_type": "InnerProduct",
      "nlinks": 32,
      "efConstruction": 100
  }

  注意: 1、向量存储只支持MemoryOnly

GPU(针对GPU编译版本):

+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+
|      字段标识      |         字段含义         |  类型  | 是否必填 |                                    备注                                     |
+====================+==========================+========+==========+=============================================================================+
| metric_type        | 计算方式                 | string | 是       | L2或者InnerProduct                                                          |
+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+
| ncentroids         | 聚类中心数量             | int    | 是       | 默认2048                                                                    |
+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+
| nsubvector         | PQ拆分子向量大小         | int    | 否       | 默认64                                                                      |
+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+
| training_threshold | 训练的数据量             | int    | 否       | 默认ncentroids * 39,是每个分片训练需要的数据量, 不是space表的数据量 |
+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+
| nprobe             | 检索时查找的聚类中心数量 | int    | 否       | 默认80                                                                      |
+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+

::
 
  "type": "GPU",
  "params": {
      "metric_type": "InnerProduct",
      "ncentroids": 2048,
      "nsubvector": 64
  }

SCANN(针对SCANN编译版本):

+--------------------+------------------+--------+----------+-----------------------------------------------------------------------------+
|      字段标识      |     字段含义     |  类型  | 是否必填 |                                    备注                                     |
+====================+==================+========+==========+=============================================================================+
| metric_type        | 计算方式         | string | 是       | L2或者InnerProduct                                                          |
+--------------------+------------------+--------+----------+-----------------------------------------------------------------------------+
| ncentroids         | 聚类中心数量     | int    | 是       | 默认2048                                                                    |
+--------------------+------------------+--------+----------+-----------------------------------------------------------------------------+
| nsubvector         | PQ拆分子向量大小 | int    | 是       | 默认128, 量化为4bit, 建议使用ivfpq模型nsubvector的2倍                       |
+--------------------+------------------+--------+----------+-----------------------------------------------------------------------------+
| thread_num         | 线程池线程数     | int    | 否       | 可以不使用, 如果使用建议为cpu核数                                           |
+--------------------+------------------+--------+----------+-----------------------------------------------------------------------------+
| training_threshold | 训练的数据量     | int    | 否       | 默认ncentroids * 39,是每个分片训练需要的数据量, 不是space表的数据量 |
+--------------------+------------------+--------+----------+-----------------------------------------------------------------------------+

::

  "type": "VEARCH",
  "params": {
      "metric_type": "InnerProduct",
      "ncentroids": 2048,
      "nsubvector": 64,
      "thread_num": 8
  }

  注意: 1、目前scann模型, 索引不支持dump/load; 不支持update。

IVFFLAT:

+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+
|      字段标识      |         字段含义         |  类型  | 是否必填 |                                    备注                                     |
+====================+==========================+========+==========+=============================================================================+
| metric_type        | 计算方式                 | string | 是       | L2或者InnerProduct                                                          |
+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+
| ncentroids         | 聚类中心数量             | int    | 是       | 默认2048                                                                    |
+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+
| training_threshold | 训练的数据量             | int    | 否       | 默认ncentroids * 39,是每个分片训练需要的数据量, 不是space表的数据量 |
+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+
| nprobe             | 检索时查找的聚类中心数量 | int    | 否       | 默认80                                                                      |
+--------------------+--------------------------+--------+----------+-----------------------------------------------------------------------------+

::
 
  "type": "IVFFLAT",
  "params": {
      "metric_type": "InnerProduct",
      "ncentroids": 2048
  }
  
 注意: 1、向量存储方式只支持RocksDB

BINARYIVF:

+--------------------+--------------------------+------+----------+---------------------------------------------------------------------+
|      字段标识      |         字段含义         | 类型 | 是否必填 |                                备注                                 |
+====================+==========================+======+==========+=====================================================================+
| ncentroids         | 聚类中心数量             | int  | 是       | 默认2048                                                            |
+--------------------+--------------------------+------+----------+---------------------------------------------------------------------+
| training_threshold | 训练的数据量             | int  | 否       | 默认ncentroids * 39,是每个分片训练需要的数据量, 不是space表的数据量 |
+--------------------+--------------------------+------+----------+---------------------------------------------------------------------+
| nprobe             | 检索时查找的聚类中心数量 | int  | 否       | 默认80                                                              |
+--------------------+--------------------------+------+----------+---------------------------------------------------------------------+

::
 
  "type": "BINARYIVF",
  "params": {
      "ncentroids": 2048
  }
  
  注意: 1、向量长度是8的倍数

FLAT:

+---------------+------------------+------------+------------+----------------------------------------+
|字段标识       |字段含义          |类型        |是否必填    |备注                                    |
+===============+==================+============+============+========================================+
|metric_type    |计算方式          |string      |是          |L2或者InnerProduct                      |
+---------------+------------------+------------+------------+----------------------------------------+

::
 
  "type": "FLAT",
  "params": {
      "metric_type": "InnerProduct"
  }
  
 注意: 1、向量存储方式只支持MemoryOnly


fields配置:

1、表空间结构定义字段支持的类型(即type的值)有8种: string(keyword), stringArray, integer,  long,  float, double,  date, vector。

2、string类型或者stringArray字段支持index属性, index定义是否创建索引, 创建索引后支持term过滤。

3、integer, long, float, double, date类型的字段支持index属性, index设置创建索引后支持数值范围过滤查询(range)。

4、vector 类型字段为特征字段, 一个表空间中支持多个特征字段, vector类型的字段支持的属性如下:

+-------------+---------------+---------------+----------+----------------------------------------------+
|字段标识     |字段含义       |类型           |是否必填  |备注                                          | 
+=============+===============+===============+==========+==============================================+
|dimension    |特征维数       |int            |是        |                                              |
+-------------+---------------+---------------+----------+----------------------------------------------+
|format       |归一化处理     |string         |否        |设置为normalization对添加的特征向量归一化处理 |
+-------------+---------------+---------------+----------+----------------------------------------------+
|store_type   |特征存储类型   |string         |否        |支持MemoryOnly、RocksDB, 不同索引默认值不一样 |
+-------------+---------------+---------------+----------+----------------------------------------------+
|store_param  |存储参数设置   |json           |否        |针对不同store_type的存储参数                  |
+-------------+---------------+---------------+----------+----------------------------------------------+
|model_id     |特征插件模型   |string         |否        |使用特征插件服务时指定                        |
+-------------+---------------+---------------+----------+----------------------------------------------+

5、dimension 定义type是vector的字段, 指定特征维数大小。

6、store_type 特征向量存储类型, 有以下几个选项: 

"MemoryOnly": 原始向量都存储在内存中, 存储数量的多少受内存限制, 适用于数据量不大(千万级), 对性能要求高的场景

"RocksDB": 原始向量存储在RockDB(磁盘)中, 存储数量受磁盘大小限制, 适用单机数据量巨大(亿级以上), 对性能要求不高的场景


7、store_param 针对不同store_type的存储参数, 其包含以下两个子参数。

cache_size: 数值类型, 单位是M bytes, 默认1024。store_type="RocksDB"时, 表示RocksDB的读缓冲大小, 值越大读向量的性能越好, 一般设置1024、2048、4096和6144即可; store_type="MemoryOnly", cache_size不生效。


标量索引

标量索引提供对标量数据的过滤功能, 开启方式参考“fields配置”中的第2条和第3条, 检索方式参考“查询”中的“filter json结构说明”


自定义表空间分片规则
>>>>>>>>>>>>>>>>>>>>>>

创建表空间时指定自定义分片规则，当数据存在过期需求，可以通过此方式直接淘汰对应分片组的数据

::
   
   curl -XPOST -H "content-type: application/json" -u "root:secret" -d'
    {
        "name": "space1",
        "partition_num": 1,
        "replica_num": 1,
        "fields": [
            {
                "name": "field_string",
                "type": "string"
            },
            {
                "name": "field_int",
                "type": "integer"
            },
            {
                "name": "field_float",
                "type": "float",
                "index": {
                    "name": "field_float",
                    "type": "SCALAR"
                }
            },
            {
                "name": "field_string_array",
                "type": "stringArray",
                "index": {
                    "name": "field_string_array",
                    "type": "SCALAR"
                }
            },
            {
                "name": "field_int_index",
                "type": "integer",
                "index": {
                    "name": "field_int_index",
                    "type": "SCALAR"
                }
            },
            {
                "name": "field_date",
                "type": "date",
                "index": {
                    "name": "field_date",
                    "type": "SCALAR"
                }
            },
            {
                "name": "field_vector",
                "type": "vector",
                "dimension": 128,
                "index": {
                    "name": "gamma",
                    "type": "IVFPQ",
                    "params": {
                        "metric_type": "InnerProduct",
                        "ncentroids": 2048,
                        "nlinks": 32,
                        "efConstruction": 40
                    }
                }
            }
        ],
        "partition_rule": {
            "type": "RANGE",
            "field": "field_date",
            "ranges": [
                {
                    "name": "p0",
                    "value": "2024-07-15"
                },
                {
                    "name": "p1",
                    "value": "2024-07-16"
                },
                {
                    "name": "p2",
                    "value": "2024-07-17"
                }
            ]
        }
    }
   ' http://${VEARCH_URL}/dbs/$db_name/spaces

partition rule详细格式: 

+----------+----------------+--------+----------+-------------------------------------------------------------------------------------------------------------------------+
| 字段标识 |    字段含义    |  类型  | 是否必须 |                                                          备注                                                           |
+==========+================+========+==========+=========================================================================================================================+
| type     | 自定义分片类型 | string | 是       | 目前只支持 RANGE                                                                                                        |
+----------+----------------+--------+----------+-------------------------------------------------------------------------------------------------------------------------+
| field    | 自定义分片字段 | string | 是       | 需要是fields里面的字段，目前只支持date类型，和_id组成共同主键，即_id相同但是partition field对应的值不同是两条不同的数据 |
+----------+----------------+--------+----------+-------------------------------------------------------------------------------------------------------------------------+
| ranges   | 自定义分片范围 | json   | 是       | 每个分片field值对应的范围,name为分片组的名称，value是对应的值的阈值且需要是递增的                                       |
+----------+----------------+--------+----------+-------------------------------------------------------------------------------------------------------------------------+

date类型目前只支持到秒级，支持两种方式传入：日期格式的字符串和时间戳，为保证数据的准确性需传入秒级的时间戳，不然内部转换会导致数据准确度丢失。

每个range会对应一个分片组，分片组的分片数量为partition_num，即设置了partition rule的表空间总共会有 len(ranges) * partition_num * replica_num 个分片, 其中len(ranges) * partition_num 个分片为完整数据。

range 范围规则为less than，不包含边界。如 “2024-07-17”则会按照 “2024-07-17 00：00：00”进行比较，这个分片组对应的就是小于“2024-07-17 00：00：00”的值，不包含“2024-07-17 00：00：00”。

删除分片组
>>>>>>>>>>>>

::
   
   curl -XPUT -H "content-type: application/json" -d'
    {
        "partition_name": "p0",
        "operator_type": "DROP"
    }
   ' http://${VEARCH_URL}/dbs/$db_name/spaces/$space_name


添加分片组
>>>>>>>>>>>>

::
   
   curl -XPUT -H "content-type: application/json" -d'
    {
        "operator_type": "ADD",
        "partition_rule": {
            "type": "RANGE",
            "ranges": [
                {
                    "name": "p3",
                    "value": "2024-07-18"
                },
                {
                    "name": "p4",
                    "value": "2024-07-19"
                }
            ]
        }
    }
   ' http://${VEARCH_URL}/dbs/$db_name/spaces/$space_name

查看表空间
------------
::
  
  curl -XGET http://${VEARCH_URL}/dbs/$db_name/spaces/$space_name

返回数据详细格式: 

+----------+----------+--------+--------------+------+
| 字段标识 | 字段含义 |  类型  | 是否一定返回 | 备注 |
+==========+==========+========+==============+======+
| code     | 返回码   | int    | 是           |      |
+----------+----------+--------+--------------+------+
| msg      | 返回信息 | string | 否           |      |
+----------+----------+--------+--------------+------+
| data     | 返回数据 | json   | 否           |      |
+----------+----------+--------+--------------+------+

data字段详细信息:

+---------------+----------------+------------+--------------+---------------------------------------+
|   字段标识    |    字段含义    |    类型    | 是否一定返回 |                 备注                  |
+===============+================+============+==============+=======================================+
| space_name    | 表名           | string     | 是           |                                       |
+---------------+----------------+------------+--------------+---------------------------------------+
| db_name       | 库名           | string     | 是           |                                       |
+---------------+----------------+------------+--------------+---------------------------------------+
| doc_num       | 表文档数量     | uint64     | 是           |                                       |
+---------------+----------------+------------+--------------+---------------------------------------+
| partition_num | 分片数量       | int        | 是           | 对表所有数据进行分片                  |
+---------------+----------------+------------+--------------+---------------------------------------+
| replica_num   | 副本数量       | int        | 是           | 通常设置为3来实现高可用               |
+---------------+----------------+------------+--------------+---------------------------------------+
| schema        | 表结构         | json       | 是           |                                       |
+---------------+----------------+------------+--------------+---------------------------------------+
| status        | 表状态         | string     | 是           | red表示表有异常,green正常, yellow预警 |
+---------------+----------------+------------+--------------+---------------------------------------+
| partitions    | 表分片详细信息 | json       | 是           | 参数控制是否返回更多详细信息          |
+---------------+----------------+------------+--------------+---------------------------------------+
| errors        | 表异常信息     | string数组 | 否           |                                       |
+---------------+----------------+------------+--------------+---------------------------------------+

返回值格式如下:
::

    {
        "code": 0,
        "data": {
            "space_name": "ts_space",
            "db_name": "ts_db",
            "doc_num": 0,
            "partition_num": 1,
            "replica_num": 3,
            "schema": {
                "fields": [
                    {
                        "name": "field_string",
                        "type": "string"
                    },
                    {
                        "name": "field_int",
                        "type": "integer"
                    },
                    {
                        "name": "field_float",
                        "type": "float",
                        "index": {
                            "name": "field_float",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_string_array",
                        "type": "stringArray",
                        "index": {
                            "name": "field_string_array",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_int_index",
                        "type": "integer",
                        "index": {
                            "name": "field_int_index",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_vector",
                        "type": "vector",
                        "dimension": 128,
                        "index": {
                            "name": "gamma",
                            "type": "IVFPQ",
                            "params": {
                                "metric_type": "InnerProduct",
                                "ncentroids": 2048,
                                "nlinks": 32,
                                "efConstruction": 40
                            }
                        }
                    }
                ]
            },
            "status": "green",
            "partitions": [
                {
                    "pid": 1,
                    "replica_num": 1,
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 1,
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                },
                {
                    "pid": 2,
                    "replica_num": 1,
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 2,
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                },
                {
                    "pid": 3,
                    "replica_num": 1,
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 3,
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                }
            ],
        }
    }

查看表partitions更多详细信息
::
  
  curl -XGET http://${VEARCH_URL}/dbs/$db_name/spaces/$space_name?detail=true

返回值格式如下:
::

    {
        "code": 0,
        "data": {
            "space_name": "ts_space",
            "db_name": "ts_db",
            "doc_num": 0,
            "partition_num": 1,
            "replica_num": 3,
            "schema": {
                "fields": [
                    {
                        "name": "field_string",
                        "type": "string"
                    },
                    {
                        "name": "field_int",
                        "type": "integer"
                    },
                    {
                        "name": "field_float",
                        "type": "float",
                        "index": {
                            "name": "field_float",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_string_array",
                        "type": "stringArray",
                        "index": {
                            "name": "field_string_array",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_int_index",
                        "type": "integer",
                        "index": {
                            "name": "field_int_index",
                            "type": "SCALAR"
                        }
                    },
                    {
                        "name": "field_vector",
                        "type": "vector",
                        "dimension": 128,
                        "index": {
                            "name": "gamma",
                            "type": "IVFPQ",
                            "params": {
                                "metric_type": "InnerProduct",
                                "ncentroids": 2048,
                                "nlinks": 32,
                                "efConstruction": 40
                            }
                        }
                    }
                ]
            },
            "status": "green",
            "partitions": [
                {
                    "pid": 1,
                    "replica_num": 1,
                    "path": "/export/Data/datas/",
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 1,
                    "raft_status": {
                        "ID": 1,
                        "NodeID": 1,
                        "Leader": 1,
                        "Term": 1,
                        "Index": 1,
                        "Commit": 1,
                        "Applied": 1,
                        "Vote": 1,
                        "PendQueue": 0,
                        "RecvQueue": 0,
                        "AppQueue": 0,
                        "Stopped": false,
                        "RestoringSnapshot": false,
                        "State": "StateLeader",
                        "Replicas": {
                            "1": {
                                "Match": 1,
                                "Commit": 1,
                                "Next": 2,
                                "State": "ReplicaStateProbe",
                                "Snapshoting": false,
                                "Paused": false,
                                "Active": true,
                                "LastActive": "2024-03-18T09: 59: 17.095112556+08: 00",
                                "Inflight": 0
                            }
                        }
                    },
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                },
                {
                    "pid": 2,
                    "replica_num": 1,
                    "path": "/export/Data/datas/",
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 2,
                    "raft_status": {
                        "ID": 2,
                        "NodeID": 1,
                        "Leader": 1,
                        "Term": 1,
                        "Index": 1,
                        "Commit": 1,
                        "Applied": 1,
                        "Vote": 1,
                        "PendQueue": 0,
                        "RecvQueue": 0,
                        "AppQueue": 0,
                        "Stopped": false,
                        "RestoringSnapshot": false,
                        "State": "StateLeader",
                        "Replicas": {
                            "1": {
                                "Match": 1,
                                "Commit": 1,
                                "Next": 2,
                                "State": "ReplicaStateProbe",
                                "Snapshoting": false,
                                "Paused": false,
                                "Active": true,
                                "LastActive": "2024-03-18T09: 59: 17.095112556+08: 00",
                                "Inflight": 0
                            }
                        }
                    },
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                },
                {
                    "pid": 3,
                    "replica_num": 1,
                    "path": "/export/Data/datas/",
                    "status": 4,
                    "color": "green",
                    "ip": "x.x.x.x",
                    "node_id": 3,
                    "raft_status": {
                        "ID": 3,
                        "NodeID": 1,
                        "Leader": 1,
                        "Term": 1,
                        "Index": 1,
                        "Commit": 1,
                        "Applied": 1,
                        "Vote": 1,
                        "PendQueue": 0,
                        "RecvQueue": 0,
                        "AppQueue": 0,
                        "Stopped": false,
                        "RestoringSnapshot": false,
                        "State": "StateLeader",
                        "Replicas": {
                            "1": {
                                "Match": 1,
                                "Commit": 1,
                                "Next": 2,
                                "State": "ReplicaStateProbe",
                                "Snapshoting": false,
                                "Paused": false,
                                "Active": true,
                                "LastActive": "2024-03-18T09: 59: 17.095112556+08: 00",
                                "Inflight": 0
                            }
                        }
                    },
                    "index_status": 0,
                    "index_num": 0,
                    "max_docid": -1
                }
            ]
        }
    }

删除表空间
----------------
::
 
  curl -XDELETE http://${VEARCH_URL}/dbs/$db_name/spaces/$space_name
