数据操作
=================
此文档使用于v3.4.2及以后版本，之前版本请参考v3.4.1版本的文档

http://router_server 代表router服务，$db_name是创建的库名, $space_name是创建的空间名, $id是数据记录的唯一id.
_id 是服务端生成的记录唯一标识，可以由用户指定，对数据的修改和删除需要使用该唯一标识。
$id 是插入数据时使用指定的值替换服务端生成的唯一标识，$id值不能使用url路径等特殊字符。若库中已存在该唯一标识的记录则更新覆盖。

插入 upsert接口
--------
如果设置了主键_id，则将使用指定的主键。 如果未设置，则由 Vearch 生成。 如果插入时指定的_id已经存在，则更新现有数据； 否则，它将被插入。
当插入数据中的documents包含多条数据，则为批量插入，一般建议批量插入不超过100条。
插入和更新现在已经支持只传入部分字段的值，插入时只传入部分字段则必须包含向量字段，更新时无此限制。

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
            "field_vector": {
                "feature": [...]
            }
        }, {
            "field_int": 45085,
            "field_float": 45085,
            "field_double": 45085,
            "field_string": "106085",
            "field_vector": {
                "feature": [...]
            }
        }, {
            "field_int": 52968,
            "field_float": 52968,
            "field_double": 52968,
            "field_string": "113968",
            "field_vector": {
                "feature": [...]
            }
        }]
    }
    ' http://router_server/document/upsert

field_vector是特征字段，其它字段为标量字段。所有字段名、值类型和定义表结构时保持一致。

插入时指定唯一标识
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "documents": [{
            "_id": 1000000,
            "field_int": 90399,
            "field_float": 90399,
            "field_double": 90399,
            "field_string": "111399",
            "field_vector": {
                "feature": [...]
            }
        }, {
            "_id": 1000001,
            "field_int": 45085,
            "field_float": 45085,
            "field_double": 45085,
            "field_string": "106085",
            "field_vector": {
                "feature": [...]
            }
        }, {
            "_id": 1000002,
            "field_int": 52968,
            "field_float": 52968,
            "field_double": 52968,
            "field_string": "113968",
            "field_vector": {
                "feature": [...]
            }
        }]
    }
    ' http://router_server/document/upsert


upsert接口返回值格式如下:
::

    {
        'code': 0,
        'msg': 'success',
        'total': 3,
        'document_ids': [{
            '_id': '-526059949411103803',
            'status': 200,
            'error': 'success'
        }, {
            '_id': '1287805132970120733',
            'status': 200,
            'error': 'success'
        }, {
            '_id': '-1948185285365684656',
            'status': 200,
            'error': 'success'
        }]
    }
total 标识插入成功的数量，document_ids返回生成的_id和插入结果信息。

精确查找 query接口
--------
/document/query 接口用于精确查找与查询条件完全匹配的数据，查找时不包含向量数据。
支持两种方式：一种是直接通过主键获取文档，另一种是根据过滤条件获取对应的文档。 
如果设置了partition_id，则获取指定数据分区上对应的文档。 此时document_id的含义就是该分区上的文档编号。 
document_id可以是指定分区的[0, max_docid]，max_docid和分区信息可以通过cluster/health接口获取。 
可以通过这种方式获取集群的完整数据。

根据唯一id标识查找数据
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "query": {
            "document_ids": ["6560995651113580768", "-5621139761924822824", "-104688682735192253"]
        },
        "vector_value": true
    }
    ' http://router_server/document/query

获取指定数据分区上对应的文档，此时document_id可以是指定分区的[0, max_docid]
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "query": {
            "document_ids": [
            "10000",
            "10001",
            "10002"
            ],
            "partition_id": "1"
        },
        "vector_value": true
    }
    ' http://router_server/document/query

根据自定义的标量字段的 Filter 表达式查找
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "query": {
            "filter": [
            {
                "range": {
                "field_int": {
                    "gte": 1000,
                    "lte": 100000
                }
                }
            },
            {
                "term": {
                "field_string": [
                    "322"
                ]
                }
            }
            ]
        },
        "vector_value": false
    }
    ' http://router_server/document/query

query接口返回格式
::

    {
        'code': 0,
        'msg': 'success',
        'total': 3,
        'documents': [{
            '_id': '6560995651113580768',
            '_source': {
                'field_double': 202558,
                'field_float': 102558,
                'field_int': 1558,
                'field_string': '1558'
            }
        }, {
            '_id': '-5621139761924822824',
            '_source': {
                'field_double': 210887,
                'field_float': 110887,
                'field_int': 89887,
                'field_string': '89887'
            }
        }, {
            '_id': '-104688682735192253',
            '_source': {
                'field_double': 207588,
                'field_float': 107588,
                'field_int': 46588,
                'field_string': '46588'
            }
        }]
    }

模糊查询 search接口
--------
支持根据指定 id 或向量数值进行相似度检索，返回指定的 Top K 个最相似的 Document。
支持根据主键 id（Document ID）或向量数值，搭配自定义的标量字段的 Filter 表达式一并进行相似度检索。
document_ids传入唯一记录id，后台处理首先根据唯一id查询出该记录的特征，然后再用特征进行相似查询，返回匹配结果。

根据document_ids 查询
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "query": {
            "document_ids": [
            "3646866681750952826"
            ],
            "filter": [
            {
                "range": {
                "field_int": {
                    "gte": 1000,
                    "lte": 100000
                }
                }
            }
            ]
        },
        "retrieval_param": {
            "metric_type": "L2"
        },
        "size": 3,
        "db_name": "ts_db",
        "space_name": "ts_space"
    }
    ' http://router_server/document/search

根据向量查询
支持单条或者多条查询，多条可以将多个查询的特征拼接成一个特征数组（比如定义128维的特征，批量查询10条，
则将10个128维特征按顺序拼接成1280维特征数组赋值给feature字段），
后台接收到请求后按表结构定义的特征字段维度进行拆分，按顺序返回匹配结果。
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "query": {
            "vector": [
            {
                "field": "field_vector",
                "feature": [
                "..."
                ]
            }
            ],
            "filter": [
            {
                "range": {
                "field_int": {
                    "gte": 1000,
                    "lte": 100000
                }
                }
            }
            ]
        },
        "retrieval_param": {
            "metric_type": "L2"
        },
        "size": 3,
        "db_name": "ts_db",
        "space_name": "ts_space"
    }
    ' http://router_server/document/search


多向量查询
表空间定义时支持多个特征字段，因此查询时可以支持相应数据的特征进行查询。以每条记录两个向量为例：定义表结构字段
::

    {
        "field1": {
            "type": "vector",
            "dimension": 128
        },
        "field2": {
            "type": "vector",
            "dimension": 256
        } 
    }


field1、field2均为向量字段，查询时搜索条件可以指定两个向量：
::

    {
        "query": {
            "vector": [{
                "field": "filed1",
                "feature": [0.1, 0.2, 0.3, 0.4, 0.5],
                "min_score": 0.9
            },
            {
                "field": "filed2",
                "feature": [0.8, 0.9],
                "min_score": 0.8
            }]
        }
    }


field1和field2过滤的结果求交集，其他参数及请求地址和普通查询一致。 

search接口返回格式
::

    {
        'code': 0,
        'msg': 'success',
        'documents': [
            [{
                '_id': '6979025510302030694',
                '_score': 16.55717658996582,
                '_source': {
                    'field_double': 207598,
                    'field_float': 107598,
                    'field_int': 6598,
                    'field_string': '6598'
                }
            }, {
                '_id': '-104688682735192253',
                '_score': 17.663991928100586,
                '_source': {
                    'field_double': 207588,
                    'field_float': 107588,
                    'field_int': 46588,
                    'field_string': '46588'
                }
            }, {
                '_id': '8549822044854277588',
                '_score': 17.88829803466797,
                '_source': {
                    'field_double': 220413,
                    'field_float': 120413,
                    'field_int': 99413,
                    'field_string': '99413'
                }
            }]
        ]
    }

查询参数整体json结构如下:
::

    {
        "query": {
            "vector": [],
            "filter": []
        },
        "retrieval_param": {"nprobe": 20},
        "fields": ["field1", "field2"],
        "is_brute_search": 0,
        "online_log_level": "debug",
        "quick": false,
        "vector_value": false,
        "load_balance": "leader",
        "l2_sqrt": false,
        "size": 10
    }


参数说明:

+-------------------+---------------+----------+-----------------------------------------+
|字段标识           |类型           |是否必填  |备注                                     | 
+===================+===============+==========+=========================================+
|vector             |json数组       |否        |查询特征，vector和document_ids必须包含一项 |
|document_ids       |json数组       |否        |查询特征，vector和document_ids必须包含一项 |
+-------------------+---------------+----------+-----------------------------------------+
|filter             |json数组       |否        |查询条件过滤: 数值过滤 + 标签过滤        |
+-------------------+---------------+----------+-----------------------------------------+
|fields             |json数组       |否        |指定返回那些字段, 默认只返回唯一id和分值 |
+-------------------+---------------+----------+-----------------------------------------+
|is_brute_search    |int            |否        |默认0                                    |
+-------------------+---------------+----------+-----------------------------------------+
|online_log_level   |string         |否        |值为debug， 开启打印调试日志             |
+-------------------+---------------+----------+-----------------------------------------+
|quick              |bool           |否        |默认false                                |
+-------------------+---------------+----------+-----------------------------------------+
|vector_value       |bool           |否        |默认false                                |
+-------------------+---------------+----------+-----------------------------------------+
|load_balance       |string         |否        |负载均衡算法，默认随机                  |
+-------------------+---------------+----------+-----------------------------------------+
|l2_sqrt            |bool           |否        |默认false,对l2距离计算结果开根号           |
+-------------------+---------------+----------+-----------------------------------------+
|sort               |json数组       |否        |指定字段排序(只针对匹配结果，非整体)         |
+-------------------+---------------+----------+-----------------------------------------+
|size               |int            |否        |指定返回结果数量,默认50                  |
+-------------------+---------------+----------+-----------------------------------------+

retrieval_param 参数指定模型计算时的参数，不同模型支持的参数不同，如下示例:

- metric_type: 计算类型，支持InnerProduct和L2, 默认L2。

- nprobe: 搜索桶数量。

- recall_num: 召回数量，默认等于查询参数中size的值，设置从索引中搜索数量，然后计算size个最相近的值。

- parallel_on_queries: 默认1， 搜索间并行；0代表桶间并行。

- efSearch: 图遍历的距离。

IVFPQ:
::
  
    "retrieval_param": {
        "parallel_on_queries": 1,
        "recall_num" : 100,
        "nprobe": 80,
        "metric_type": "L2" 
    }

GPU:
::
    "retrieval_param": {
        "recall_num" : 100,
        "nprobe": 80,
        "metric_type": "L2"
    }

HNSW:
::
    "retrieval_param": {
        "efSearch": 64,
        "metric_type": "L2"
    }

IVFFLAT:
::

    "retrieval_param": {
        "parallel_on_queries": 1,
        "nprobe": 80,
        "metric_type": "L2"
    }

FLAT:
::

    "retrieval_param": {
        "metric_type": "L2"
    }

- vector json结构说明:
::

    "vector": [{
                "field": "field_name",
                "feature": [0.1, 0.2, 0.3, 0.4, 0.5],
                "min_score": 0.9,
                "boost": 0.5
            }]


(1) vector 支持多个(对应定义表结构时包含多个特征字段)。

(2) field 指定创建表时特征字段的名称。

(3) feature 传递特征，维数和定义表结构时维数必须相同。

(4) min_score 指定返回结果中分值必须大于等于0.9，两个向量计算结果相似度在0-1之间，min_score可以指定返回结果分值最小值，max_score可以指定最大值。如设置： “min_score”: 0.8，“max_score”: 0.95  代表过滤0.8<= 分值<= 0.95 的结果。同时另外一种分值过滤的方式是使用: "symbol":">="，"value":0.9 这种组合方式，symbol支持的值类型包含: > 、 >= 、 <、 <=  4种，value及min_score、max_score值在0到1之间。

(5) boost指定相似度的权重，比如两个向量相似度分值是0.7，boost设置成0.5之后,返回的结果中会将分值0.7乘以0.5即0.35，当单个向量时不生效。

- filter json结构说明:
::

    "filter": [
        {
            "range": {
                "field_name": {
                    "gte": 160,
                    "lte": 180
                }
            }
        },
        {
            "term": {
                "field1": ["100", "200", "300"],
                "operator": "or"
            }
        },
        {
            "term": {
                "field2": ["a", "b", "c"],
                "operator": "and"
            }
        },
        {
            "term": {
                "field3": ["A1", "B2"],
                "operator": "not"
            } 
        }
    ]

(1) filter 条件支持多个，多个条件之间是交的关系。

(2) range 指定使用数值字段integer、long、float、double 过滤， filed_name是数值字段名称， gte、lte指定范围， lte 小于等于， gte大于等于，若使用等值过滤，lte和gte设置相同的值。上述示例表示查询field_name字段大于等于160小于等于180区间的值。

(3) term 使用标签过滤（string字段）， field1是定义的标签字段名，允许使用多个值过滤，可以求并“operator”: “or” , 求交: “operator”: “and”，不包含: "operator": "not"。

- is_brute_search  0使用索引搜索（建完索引前查询结果为空）， 1使用暴力搜索，默认值0。

- online_log_level 设置成”debug” 可以指定在服务端打印更加详细的日志，开发测试阶段方便排查问题。

- quick 搜索结果默认将PQ召回向量进行计算和精排，为了加快服务端处理速度设置成true可以指定只召回，不做计算和精排。

- vector_value 为了减小网络开销，搜索结果中默认不包含特征数据只包含标量信息字段，设置成true指定返回结果中包含原始特征数据。

- load_balance leader，random，no_leader，默认leader仅从主数据节点查询，random: 从ps主从节点随机选择，no_leader:只查询从节点，least_connection：最少连接数。

- size 指定最多返回的结果数量。若请求url中设置了size值，优先使用url中指定的size值。


删除 delete接口
--------
删除支持两种方法：指定document_ids和过滤条件。

删除指定document_ids
::

    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "query": {
            'document_ids': ['4501743250723073467', '616335952940335471', '-2422965400649882823']
        }
    }
    ' http://router_server/document/delete
  
删除满足过滤条件的文档，size指定每个数据分片删除的条数
::
  
    curl -H "content-type: application/json" -XPOST -d'
    {
        "db_name": "ts_db",
        "space_name": "ts_space",
        "query": {
            "filter": [
            {
                "range": {
                "field_int": {
                    "gte": 1000,
                    "lte": 100000
                }
                }
            },
            {
                "term": {
                "field_string": [
                    "322"
                ]
                }
            }
            ]
        },
        "size": 3
    }
    ' http://router_server/document/delete


delete接口返回格式
::

    {
        'code': 0,
        'msg': 'success',
        'total': 3,
        'document_ids': ['4501743250723073467', '616335952940335471', '-2422965400649882823']
    }

