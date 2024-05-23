数据操作
=================
http://${VEARCH_URL} 代表vearch服务, $db_name是创建的库名, $space_name是创建的空间名, $id是数据记录的唯一id.

_id 是服务端生成的记录唯一标识，可以由用户指定，对数据的修改和删除需要使用该唯一标识。

$id 是插入数据时使用指定的值替换服务端生成的唯一标识，$id值不能使用url路径等特殊字符。若库中已存在该唯一标识的记录则更新覆盖。

插入 upsert接口
----------------

如果设置了主键_id, 则将使用指定的主键。 如果未设置, 则由 Vearch 生成。 

如果插入时指定的_id已经存在, 则更新现有数据; 否则, 它将被插入。

当插入数据中的documents包含多条数据, 则为批量插入, 一般建议批量插入不超过100条。

插入和更新现在支持只传入部分字段的值, 插入时只传入部分字段必须包含向量字段, 更新时无此限制。

插入时不指定唯一标识id
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "documents": [{
            "field_int": 90399,
            "field_float": 90399,
            "field_double": 90399,
            "field_string": "111399",
            "field_vector": [...]
        }, {
            "field_int": 45085,
            "field_float": 45085,
            "field_double": 45085,
            "field_string": "106085",
            "field_vector":  [...]
        }, {
            "field_int": 52968,
            "field_float": 52968,
            "field_double": 52968,
            "field_string": "113968",
            "field_vector":  [...]
        }]
    }
    ' http://${VEARCH_URL}/document/upsert

field_vector是向量字段, 其它字段为标量字段。所有字段名、值类型和定义表结构时保持一致。

插入时指定唯一标识
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "documents": [{
            "_id": "1000000",
            "field_int": 90399,
            "field_float": 90399,
            "field_double": 90399,
            "field_string": "111399",
            "field_vector":  [...]
        }, {
            "_id": "1000001",
            "field_int": 45085,
            "field_float": 45085,
            "field_double": 45085,
            "field_string": "106085",
            "field_vector": [...]
        }, {
            "_id": "1000002",
            "field_int": 52968,
            "field_float": 52968,
            "field_double": 52968,
            "field_string": "113968",
            "field_vector": [...]
        }]
    }
    ' http://${VEARCH_URL}/document/upsert


upsert接口返回值格式如下
::

    {
        "code": 0,
        "msg": "success",
        "data": {
            "total": 3,
            "document_ids": [
                {
                    "_id": "-526059949411103803"
                },
                {
                    "_id": "1287805132970120733"
                },
                {
                    "_id": "-1948185285365684656"
                }
            ]
        }
    }

total 标识插入成功的数量, document_ids返回生成的_id和插入结果信息。

精确查找 query接口
------------------------

/document/query 接口用于精确查找与查询条件完全匹配的数据, 查找时不可包含向量数据。

支持两种方式: 一种是直接通过主键获取文档, 另一种是根据过滤条件获取对应的文档。 

如果直接通过document_ids获取文档时设置了partition_id, 则获取指定数据分区上对应的文档。 此时document_id的含义就是该分区上的文档编号。

document_id可以是指定分区的[0, max_docid], max_docid和partition_id信息可以通过 http://master_server/dbs/$db_name/spaces/$space_name 接口获取。 
可以通过这种方式获取集群的完整数据。

query 接口参数说明:

+--------------+------------+----------+---------------------------------------------------------------------+
|   字段标识   |    类型    | 是否必填 |                                备注                                 |
+==============+============+==========+=====================================================================+
| document_ids | string数组 | 否       | 查询条件, filters和document_ids必须包含一项                         |
+--------------+------------+----------+---------------------------------------------------------------------+
| partition_id | int        | 否       | 指定在哪个partition获取数据, 与document_ids结合使用                 |
+--------------+------------+----------+---------------------------------------------------------------------+
| filters      | json数组   | 否       | 查询条件过滤: 数值过滤 + 标签过滤, filter和document_ids必须包含一项 |
+--------------+------------+----------+---------------------------------------------------------------------+
| fields       | string数组 | 否       | 指定返回那些字段, 默认返回除向量字段外的所有字段                    |
+--------------+------------+----------+---------------------------------------------------------------------+
| vector_value | bool       | 否       | 默认false,是否返回向量                                              |
+--------------+------------+----------+---------------------------------------------------------------------+
| limit        | int        | 否       | 指定返回结果数量,默认50                                             |
+--------------+------------+----------+---------------------------------------------------------------------+

- filter json结构说明
::

    "filters": {
        "operator": "AND",
        "conditions": [
            {
                "field": "field_int",
                "operator": ">=",
                "value": 1
            },
            {
                "field": "field_int",
                "operator": "<=",
                "value": 3
            },
            {
                "field": "field_string",
                "operator": "IN",
                "value": ["aaa", "bbb"]
            }
        ]
    }


filters 格式说明:

+------------+----------+----------+---------------+
|  字段标识  |   类型   | 是否必填 |     备注      |
+============+==========+==========+===============+
| operator   | string   | 是       | 目前只支持AND |
+------------+----------+----------+---------------+
| conditions | json数组 | 是       | 详细过滤条件  |
+------------+----------+----------+---------------+

(1) filter 条件支持多个, 多个条件之间是交的关系, 即最外层operator目前支持AND。

conditions 格式说明:

+----------+--------+----------+-------------------------------+
| 字段标识 |  类型  | 是否必填 |             备注              |
+==========+========+==========+===============================+
| field    | string | 是       | 过滤字段名                    |
+----------+--------+----------+-------------------------------+
| operator | string | 是       | 操作符, 支持 >, >=, <, <=, IN |
+----------+--------+----------+-------------------------------+
| value    | json   | 是       | 过滤值                        |
+----------+--------+----------+-------------------------------+

(2) conditions 具体过滤条件, 目前支持两类字段类型过滤, 数值类型和字符串类型(包括字符串数组类型)
数值类型操作符: >, >=, <, <= ; 字符串操作符类型 IN

根据唯一id标识查找数据
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "document_ids": ["6560995651113580768", "-5621139761924822824", "-104688682735192253"]
        "vector_value": true
    }
    ' http://${VEARCH_URL}/document/query

获取指定数据分区上对应的文档, 此时document_id可以是指定分区的[0, max_docid]
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "document_ids": ["0", "1", "2"],
        "partition_id": 1,
        "vector_value": true
    }
    ' http://${VEARCH_URL}/document/query

根据自定义的标量字段的 Filter 表达式查找
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "filters": {
            "operator": "AND",
            "conditions": [
                {
                    "field": "field_int",
                    "operator": >=,
                    "value": 1
                },
                {
                    "field": "field_int",
                    "operator": <=,
                    "value": 3
                }
            ]
        }
    }
    ' http://${VEARCH_URL}/document/query

query接口返回格式
::

    {
        "code": 0,
        "msg": "success",
        "data": {
            "total": 3,
            "documents": [{
                "_id": "6560995651113580768",
                "field_double": 202558,
                "field_float": 102558,
                "field_int": 1558,
                "field_string": "1558"
            }, {
                "_id": "-5621139761924822824",
                "field_double": 210887,
                "field_float": 110887,
                "field_int": 89887,
                "field_string": "89887"
            }, {
                "_id": "-104688682735192253",
                "field_double": 207588,
                "field_float": 107588,
                "field_int": 46588,
                "field_string": "46588"
            }]
        }
    }

模糊查询 search接口
------------------------

根据向量数值进行相似度检索, 返回指定的 limit 个最相似的 Document。

参数说明:

+-----------------+----------+----------+-------------------------------------------------------------------+
|    字段标识     |   类型   | 是否必填 |                               备注                                |
+=================+==========+==========+===================================================================+
| vectors         | json数组 | 是       | 查询特征, vectors和document_ids必须包含一项                       |
+-----------------+----------+----------+-------------------------------------------------------------------+
| filters         | json数组 | 否       | 查询条件过滤: 数值过滤 + 标签过滤                                 |
+-----------------+----------+----------+-------------------------------------------------------------------+
| fields          | json数组 | 否       | 指定返回那些字段, 默认只返回唯一id和分值                          |
+-----------------+----------+----------+-------------------------------------------------------------------+
| is_brute_search | int      | 否       | 默认0                                                             |
+-----------------+----------+----------+-------------------------------------------------------------------+
| vector_value    | bool     | 否       | 默认false, 是否返回向量                                           |
+-----------------+----------+----------+-------------------------------------------------------------------+
| load_balance    | string   | 否       | 负载均衡算法, 默认随机                                            |
+-----------------+----------+----------+-------------------------------------------------------------------+
| limit           | int      | 否       | 指定返回结果数量,默认50                                           |
+-----------------+----------+----------+-------------------------------------------------------------------+
| ranker          | json     | 否       | 对多向量结果进一步处理, 目前只支持WeightedRanker,指定相似度的权重 |
+-----------------+----------+----------+-------------------------------------------------------------------+
| index_params    | json     | 否       | 指定模型计算时的参数                                              |
+-----------------+----------+----------+-------------------------------------------------------------------+

查询参数整体json简单示例如下:
::

    {
        "vectors": [],
        "filters": [],
        "index_params": {"nprobe": 20},
        "fields": ["field1", "field2"],
        "is_brute_search": 0,
        "vector_value": false,
        "load_balance": "leader",
        "limit": 10, 
        "ranker": {
            "type": "WeightedRanker",
            "params": [0.5, 0.5]
        }
    }

index_params 参数指定索引计算时的参数, 不同索引支持的参数不同, 如下示例:

- metric_type: 计算类型, 支持InnerProduct和L2, 默认L2。

- nprobe: 搜索桶数量。

- recall_num: 精排召回数量, 默认等于查询参数中limit的值, 设置从索引中查询到结果之后利用原始向量重新计算距离重新排序。

- parallel_on_queries: 默认1,  搜索间并行; 0代表桶间并行。

- efSearch: 图遍历的距离。

IVFPQ
::

    "index_params": {
        "parallel_on_queries": 1,
        "recall_num" : 100,
        "nprobe": 80,
        "metric_type": "L2" 
    }

    当设置recall_num会用原始向量做计算重排(精排)

GPU
::

    "index_params": {
        "recall_num" : 100,
        "nprobe": 80,
        "metric_type": "L2"
    }

HNSW
::

    "index_params": {
        "efSearch": 64,
        "metric_type": "L2"
    }

IVFFLAT
::

    "index_params": {
        "parallel_on_queries": 1,
        "nprobe": 80,
        "metric_type": "L2"
    }

FLAT
::

    "index_params": {
        "metric_type": "L2"
    }

- vectors json结构说明:
::

    "vectors": [{
                "field": "field_name",
                "feature": [0.1, 0.2, 0.3, 0.4, 0.5],
                "min_score": 0.9
            }]


(1) vector 支持多个(对应定义表结构时包含多个特征字段)。

(2) field 指定创建表时特征字段的名称。

(3) feature 传递特征, 维数和定义表结构时维数必须相同。

(4) min_score 指定返回结果中分值必须大于等于0.9, min_score可以指定返回结果分值最小值, max_score可以指定最大值。如设置:  “min_score”: 0.8, “max_score”: 0.95  代表过滤0.8<= 分值<= 0.95 的结果。同时另外一种分值过滤的方式是使用: "symbol":">=", "value":0.9 这种组合方式, symbol支持的值类型包含: > 、 >= 、 <、 <=  4种, value及min_score、max_score值在0到1之间。

- filter json结构说明:

参考query接口部分对filter json的说明

- is_brute_search  0使用索引搜索(建完索引前查询结果为空),  1使用暴力搜索, 默认值0。

- vector_value 为了减小网络开销, 搜索结果中默认不包含特征数据只包含标量信息字段, 设置成true指定返回结果中包含原始特征数据。

- load_balance leader, random, no_leader, least_connection, 默认random。leader仅从主数据节点查询, random: 从ps主从节点随机选择, no_leader:只查询从节点, least_connection: 最少连接数。

- limit 指定最多返回的结果数量。若请求url中设置了limit值, 优先使用url中指定的limit值。

根据向量查询
支持单条或者多条查询, 多条可以将多个查询的特征拼接成一个特征数组(比如定义128维的特征, 批量查询10条, 则将10个128维特征按顺序拼接成1280维特征数组赋值给feature字段), 后台接收到请求后按表结构定义的特征字段维度进行拆分, 按顺序返回匹配结果。
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "vectors": [
            {
                "field": "field_vector",
                "feature": [
                    "..."
                ]
            }
        ],
        "filters": {
            "operator": "AND",
            "conditions": [
                {
                    "field": "field_int",
                    "operator": ">=",
                    "value": 1
                },
                {
                    "field": "field_int",
                    "operator": "<=",
                    "value": 3
                },
                {
                    "field": "field_string",
                    "operator": "IN",
                    "value": [
                        "aaa",
                        "bbb"
                    ]
                }
            ]
        },
        "index_params": {
            "metric_type": "L2"
        },
        "limit": 3,
        "db_name": "ts_db",
        "space_name": "ts_space"
    }
    ' http://${VEARCH_URL}/document/search


多向量查询
表空间定义时支持多个特征字段, 因此查询时可以支持相应数据的特征进行查询。以每条记录两个向量为例: 定义表结构字段
::

    {
        "field_vector1": {
            "type": "vector",
            "dimension": 128
        },
        "field_vector2": {
            "type": "vector",
            "dimension": 256
        } 
    }


field_vector1、field_vector2均为向量字段, 查询时搜索条件可以指定两个向量:
::

    {
        "vectors": [{
            "field": "field_vector1",
            "feature": [...]
        },
        {
            "field": "field_vector2",
            "feature": [...]
        }],
        "ranker": {
            "type": "WeightedRanker",
            "params": [0.5, 0.5]
        }
    }


field1和field2过滤的结果求交集, 其他参数及请求地址和普通查询一致。 

search接口返回格式
::

    {
        "code": 0,
        "msg": "success",
        "data": {
            "documents": [
                [{
                    "_id": "6979025510302030694",
                    "_score": 16.55717658996582,
                    "field_double": 207598,
                    "field_float": 107598,
                    "field_int": 6598,
                    "field_string": "6598",
                }, {
                    "_id": "-104688682735192253",
                    "_score": 17.663991928100586,
                    "field_double": 207588,
                    "field_float": 107588,
                    "field_int": 46588,
                    "field_string": "46588"
                }, {
                    "_id": "8549822044854277588",
                    "_score": 17.88829803466797,
                    "field_double": 220413,
                    "field_float": 120413,
                    "field_int": 99413,
                    "field_string": "99413"
                }]
            ]        
        }
    }


删除 delete接口
------------------------

删除支持两种方法: 指定document_ids和过滤条件。

删除指定document_ids
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "document_ids": ["4501743250723073467", "616335952940335471", "-2422965400649882823"]
    }
    ' http://${VEARCH_URL}/document/delete
  
删除满足过滤条件的文档, limit指定每个数据分片删除的条数
::
  
    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "filters": {
            "operator": "AND",
            "conditions": [
                {
                    "field": "field_int",
                    "operator": ">=",
                    "value": 1
                },
                {
                    "field": "field_int",
                    "operator": "<=",
                    "value": 3
                }
            ]
        },
        "limit": 3
    }
    ' http://${VEARCH_URL}/document/delete


delete接口返回格式
::

    {
        "code": 0,
        "msg": "success",
        "data": {
            "total": 3,
            "document_ids": ["4501743250723073467", "616335952940335471", "-2422965400649882823"]
        }
    }

