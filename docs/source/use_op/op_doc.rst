数据操作
=================

http://router_server 代表router服务，$db_name是创建的库名, $space_name是创建的空间名, $id是数据记录的唯一id.

单条插入
--------

插入时不指定唯一标识id
::

  curl -XPOST -H "content-type: application/json"  -d'
  {
      "field1": "value1",
      "field2": "value2",
      "field3": {
          "feature": [0.1, 0.2]
      }
  }
  ' http://router_server/$db_name/$space_name

field1和field2是标量字段，field3是特征字段。所有字段名、值类型和定义表结构时保持一致。

返回值格式如下:
::

  {
      "_index": "db1",
      "_type": "space1",
      "_id": "AW5J1lNmJG6WbbCkHrFW",
      "status": 200
  }

其中_index 库名称， _type 表空间名称，_id 是服务端生成的记录唯一标识，可以由用户指定，对数据的修改和删除需要使用该唯一标识。


插入时指定唯一标识
::

  curl -XPOST -H "content-type: application/json"  -d'
  {
      "field1": "value1",
      "field2": "value2",
      "field3": {
          "feature": [0.1, 0.2]
      }
  } 
  ' http://router_server/$db_name/$space_name/$id

$id 是插入数据时使用指定的值替换服务端生成的唯一标识，$id值不能使用url路径等特殊字符。若库中已存在该唯一标识的记录则覆盖。


批量插入
--------

::

  curl -H "content-type: application/json" -XPOST -d'
  {"index": {"_id": "v1"}}\n
  {"field1": "value", "field2": {"feature": []}}\n
  {"index": {"_id": "v2"}}\n
  {"field1": "value", "field2": {"feature": []}}\n
  ' http://router_server/$db_name/$space_name/_bulk

json格式的变体，{"index": {"_id": "v1"}} 指定记录的id, _id值为空后台自动生成唯一id, {"field1": "value",  "field2": {"feature": []}} 指定插入的数据，每行json字符串均以\\n结尾。

更新
--------
更新时必须指定唯一标识id
::

  curl -H "content-type: application/json" -XPOST -d'
  {
      "field1": "value1",
      "field2": "value2",
      "field3": {
          "feature": [0.1, 0.2]
      }
  }
  ' http://router_server/$db_name/$space_name/$id/_update

请求路径中指定唯一标识$id，使用指定id插入的方式进行数据覆盖更新(后续可支持单个字段修改)。


删除
--------
根据唯一id标识删除数据
::

  curl -XDELETE http://router_server/$db_name/$space_name/$id


根据查询过滤结果删除数据
::

  curl -H "content-type: application/json" -XPOST -d'
  {
      "query": {
          "sum": [{}]
      }
  }   
  ' http://router_server/$db_name/$space_name/_delete_by_query


查询详细语法见下文

查询
--------
查询示例
::

  curl -H "content-type: application/json" -XPOST -d'
  {
      "query": {
          "sum": [{
              "field": "field_name",
              "feature": [0.1, 0.2, 0.3, 0.4, 0.5],
              "min_score": 0.9,
              "boost": 0.5
          }],
          "filter": [{
              "range": {
                  "field_name": {
                      "gte": 160,
                      "lte": 180
                  }
              }
          },
          {
               "term": {
                   "field_name": ["100", "200", "300"],
                   "operator": "or"
               }
          }]
      },
      "retrieval_param": {
          "nprobe": 20
      },
      "fields": ["field1", "field2"],
      "is_brute_search": 0,
      "online_log_level": "debug",
      "quick": false,
      "vector_value": false,
      "client_type": "leader",
      "l2_sqrt": false,
      "sort": [{"field1":{"order": "asc"}}],
      "size": 10
  }  
  ' http://router_server/$db_name/$space_name/_search


查询参数整体json结构如下:
::

  {
      "query": {
          "sum": [],
          "filter": []
      },
      "retrieval_param": {"nprobe": 20},
      "fields": ["field1", "field2"],
      "is_brute_search": 0,
      "online_log_level": "debug",
      "quick": false,
      "vector_value": false,
      "client_type": "leader",
      "l2_sqrt": false,
      "sort": [{"field1":{"order": "asc"}}],
      "size": 10
  }


参数说明:

+-------------------+---------------+----------+-----------------------------------------+
|字段标识           |类型           |是否必填  |备注                                     | 
+===================+===============+==========+=========================================+
|sum                |json数组       |是        |查询特征                                 |
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
|client_type        |string         |否        |默认leader                               |
+-------------------+---------------+----------+-----------------------------------------+
|ivf_flat           |bool           |否        |默认false,仅适用于IVFPQ模型,结果开根号   |
+-------------------+---------------+----------+-----------------------------------------+
|sort               |json数组       |否        |指定字段排序(只针对匹配结果，非整体)     |
+-------------------+---------------+----------+-----------------------------------------+
|size               |int            |否        |指定返回结果数量                         |
+-------------------+---------------+----------+-----------------------------------------+

retrieval_param 参数指定模型计算时的参数，不同模型支持的参数不同，如下示例:

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

- sum json结构说明:
::

  "sum": [{
            "field": "field_name",
            "feature": [0.1, 0.2, 0.3, 0.4, 0.5],
            "min_score": 0.9,
            "boost": 0.5
         }]


(1) sum 支持多个(对应定义表结构时包含多个特征字段)。

(2) field 指定创建表时特征字段的名称。

(3) feature 传递特征，维数和定义表结构时维数必须相同。

(4) min_score 指定返回结果中分值必须大于等于0.9，两个向量计算结果相似度在0-1之间，min_score可以指定返回结果分值最小值，max_score可以指定最大值。如设置： “min_score”: 0.8，“max_score”: 0.95  代表过滤0.8<= 分值<= 0.95 的结果。同时另外一种分值过滤的方式是使用: "symbol":">="，"value":0.9 这种组合方式，symbol支持的值类型包含: > 、 >= 、 <、 <=  4种，value及min_score、max_score值在0到1之间。

(5) boost指定相似度的权重，比如两个向量相似度分值是0.7，boost设置成0.5之后,返回的结果中会将分值0.7乘以0.5即0.35。

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
                   },
                   "term": {
                       "field2": ["a", "b", "c"],
                       "operator": "and"
                   },
                   "term": {
                       "field3": ["A1", "B2"],
                       "operator": "not"
                   } 
               }
            ]

(1) filter 条件支持多个，多个条件之间是交的关系。

(2) range 指定使用数值字段integer/float 过滤， filed_name是数值字段名称， gte、lte指定范围， lte 小于等于， gte大于等于，若使用等值过滤，lte和gte设置相同的值。上述示例表示查询field_name字段大于等于160小于等于180区间的值。

(3) term 使用标签过滤， field1是定义的标签字段，允许使用多个值过滤，可以求并“operator”: “or” , 求交: “operator”: “and”，不包含: "operator": "not"。

- is_brute_search 0代表若特征已经创建索引则使用索引，否则暴力搜索； -1 代表只使用索引进行搜索， 1代表不使用索引只进行暴力搜索。默认值0。

- online_log_level 设置成”debug” 可以指定在服务端打印更加详细的日志，开发测试阶段方便排查问题。

- quick 搜索结果默认将PQ召回向量进行计算和精排，为了加快服务端处理速度设置成true可以指定只召回，不做计算和精排。

- vector_value 为了减小网络开销，搜索结果中默认不包含特征数据只包含标量信息字段，设置成true指定返回结果中包含原始特征数据。

- client_type leader，random，no_leader，默认leader仅从主数据节点查询，random: 从ps主从节点随机选择，no_leader:只查询从节点。

- size 指定最多返回的结果数量。若请求url中设置了size值http://router_server/$db_name/$space_name/_search?size=20优先使用url中指定的size值。


id查询
--------
::

  curl -XGET http://router_server/$db_name/$space_name/$id
  
批量id查询
--------
::
  
  curl -H "content-type: application/json" -XPOST -d'
  {
      "query": {
	        "ids": ["id1", "id2"],
	        "fields": ["field1"]
      }
  }
  ' http://router_server/$db_name/$space_name/_query_byids


ids指定多个id， fields 指定返回每条记录中那些字段


批量特征查询1
--------
::
 
  curl -H "content-type: application/json" -XPOST -d'
  [{
     "query": {
         "sum": [{
             "field": "vector_field_name",
             "feature": [0.1, 0.2]
         }]
     }
  },
  {
     "query": {
         "sum": [{
             "field": "vector_field_name",
             "feature": [0.1, 0.2]
         }]
      }
  }]
  ' http://router_server/$db_name/$space_name/_bulk_search
 
把多个单条查询的参数拼接成数组作为请求参数，返回结果和请求顺序保持一致。


批量特征查询2
--------
::

  curl -H "content-type: application/json" -XPOST -d'
  {
      "query": {
          "sum": [{
              "field": "vector_field_name",
              "feature": [0.1, 0.2]
          }]
      }
  }
  ' http://router_server/$db_name/$space_name/_msearch

适用于多个查询使用相同过滤条件的情况，将多个查询的特征拼接成一个特征数组（比如定义128维的特征，批量查询10条，则将10个128维特征按顺序拼接成1280维特征数组赋值给feature字段），后台接收到请求后按表结构定义的特征字段维度进行拆分，按顺序返回匹配结果。


根据id特征查询
--------
::

  curl -H "content-type: application/json" -XPOST -d'
  {
      "query": {
          "ids": ["id1", "id2"]
       },
       "size": 10
  }
  ' http://router_server/$db_name/$space_name/_query_byids_feature
   
传入记录id， 首先查询出该记录的特征，然后再用特征进行查询，返回匹配结果。


多向量查询
--------
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
          "sum": [{
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

