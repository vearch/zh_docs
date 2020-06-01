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
      "status": 201,
      "_version": 1,
      "_shards": {
          "total": 0,
          "successful": 1,
          "failed": 0
      },
      "result": "created",
      "_seq_no": 1,
      "_primary_term": 1
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

根据id批量删除
::

    curl -H "content-type: application/json" -XPOST -d'
    {"delete": {"_id": "v1"}}
    {"delete": {"_id": "v2"}}
    {"delete": {"_id": "v3"}}
    ' http://router_server/$db_name/$space_name/_bulk


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
      "fields": ["field1", "field2"],
      "direct_search_type": 0,
      "online_log_level": "debug",
      "quick": false,
      "vector_value": false,
      "client_type": "leader",
      "parallel": false,
      "l2_sqrt": false,
      "ivf_flat": false,
      "nprobe": 20,
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
      "fields": ["field1", "field2"],
      "direct_search_type": 0,
      "online_log_level": "debug",
      "quick": false,
      "vector_value": false,
      "client_type": "leader",
      "parallel": false,
      "l2_sqrt": false,
      "ivf_flat": false,
      "nprobe": 20,
      "size": 10
  }


参数说明:

+-------------------+---------------+----------+----------------------------------+
|字段标识           |类型           |是否必填  |备注                              | 
+===================+===============+==========+==================================+
|sum                |json数组       |是        |查询特征                          |
+-------------------+---------------+----------+----------------------------------+
|filter             |json数组       |否        |查询条件过滤: 数值过滤 + 标签过滤 |
+-------------------+---------------+----------+----------------------------------+
|fields             |json数组       |否        |指定返回那些字段, 默认所有字段    |
+-------------------+---------------+----------+----------------------------------+
|direct_search_type |int            |否        |默认0                             |
+-------------------+---------------+----------+----------------------------------+
|online_log_level   |string         |否        |值为debug， 开启打印调试日志      |
+-------------------+---------------+----------+----------------------------------+
|quick              |bool           |否        |默认false                         |
+-------------------+---------------+----------+----------------------------------+
|vector_value       |bool           |否        |默认false                         |
+-------------------+---------------+----------+----------------------------------+
|client_type        |string         |否        |默认leader                        |
+-------------------+---------------+----------+----------------------------------+
|parallel           |bool           |否        |默认false                         |
+-------------------+---------------+----------+----------------------------------+
|l2_sqrt            |bool           |否        |默认false                         |
+-------------------+---------------+----------+----------------------------------+
|ivf_flat           |bool           |否        |默认false,仅适用于IVFPQ模型       |
+-------------------+---------------+----------+----------------------------------+
|nprobe             |int            |否        |默认20,搜索聚类中心桶数量         |
+-------------------+---------------+----------+----------------------------------+
|size               |int            |否        |指定返回结果数量                  |
+-------------------+---------------+----------+----------------------------------+

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
                       "field_name": ["100", "200", "300"],
                       "operator": "or"
                   }
               }
            ]

(1) filter 条件支持多个，多个条件之间是交的关系。

(2) range 指定使用数值字段integer/float 过滤， filed_name是数值字段名称， gte、lte指定范围， lte 小于等于， gte大于等于，若使用等值过滤，lte和gte设置相同的值。上述示例表示查询field_name字段大于等于160小于等于180区间的值。

(3) term 使用标签过滤， field_name是定义的标签字段，允许使用多个值过滤，可以求并“operator”: “or” , 求交: “operator”: “and”，上述示例表示查询field_name字段值是”100”、”200” 或”300”的值。

- direct_search_type 指定查询类型，0代表若特征已经创建索引则使用索引，若没有创建则暴力搜索； -1 代表只使用索引进行搜索， 1代表不使用索引只进行暴力搜索。默认值是0。

- online_log_level 设置成”debug” 可以指定在服务端打印更加详细的日志，开发测试阶段方便排查问题。

- quick 搜索结果默认将PQ召回向量进行计算和精排，为了加快服务端处理速度设置成true可以指定只召回，不做计算和精排。

- vector_value 为了减小网络开销，搜索结果中默认不包含特征数据只包含标量信息字段，设置成true指定返回结果中包含原始特征数据。

- client_type leader，random，no_leader，默认leader仅从主数据节点查询，random: 从ps主从节点随机选择，no_leader:只查询从节点。

- parallel 设置为true适合批量查询，默认false。

- l2_sqrt default false， don't do sqrt; TRUE, do sqrt。 

- ivf_flat 仅适用与IVFPQ模型，默认false，设置为true，使用ivf_flat替换ivfpq。

- nprobe 指定搜索聚类中心数量，适用与IVFPQ和GPU模型，默认20

- size 指定最多返回的结果数量。若请求url中设置了size值http://router_server/$db_name/$space_name/_search?size=20优先使用url中指定的size值。


id查询
--------
::

  curl -XGET http://router_server/$db_name/$space_name/$id
 

批量查询
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

批量查询和单条查询的区别在于将批量的特征按顺序拼接成一个特征数组，后台服务会按照定义表空间结构时特征维数进行拆分。比如定义10维的特征字段，批量50条进行查询，将特征按顺序拼接成500维的数组赋值给feature参数。请求后缀使用_msearch。


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

