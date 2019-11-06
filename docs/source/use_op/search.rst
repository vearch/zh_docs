查询
==================

xxxxxx 是router服务域名或ip:port

$db_name, $space_name是查询的库表名称

只使用特征查询(size=10指定最多返回10条记录)
::
  curl -XPOST -H "content-type: application/json" -d'
  {
    "query":{
      "sum" : [
        {
           "field": "feature1",
           "feature": [0, 0, 0, 0, 0, 0],
        }
      ]
    }
  }
  ' http://xxxxxx/$db_name/$space_name/_search?size=10


查询限制分值(如下示例过滤分值大于等于0.9)
::
  curl -XPOST -H "content-type: application/json" -d'
  {
    "query":{
      "sum" : [
        {
           "field": "feature1",
           "feature": [0, 0, 0, 0, 0, 0],
           "min_score": 0.9
        }
      ]
    }
  }
  ' http://xxxxxx/$db_name/$space_name/_search?size=10


使用数值字段过滤
::
  curl -XPOST -H "content-type: application/json" -d'
  {
    "query":{
      "sum" : [
        {
           "field": "feature1",
           "feature": [0, 0, 0, 0, 0, 0],
           "min_score": 0.9
        }
      ],
      "filter": [
        {
          "range": {
            "number_filed": {
               "gte" : 1,
               "lte" : 3
            }
          }
        }
      ]
    }
  }
  ' http://xxxxxx/$db_name/$space_name/_search?size=10

使用条件过滤
::
  curl -XPOST -H "content-type: application/json" -d'
  {
    "query":{
      "sum" : [
        {
           "field": "feature1",
           "feature": [0, 0, 0, 0, 0, 0],
           "min_score": 0.9
        }
      ],
      "filter": [
        {
          "range": {
            "number_filed": {
               "gte" : 1,
               "lte" : 3
            }
          },
          "term": {
            "tags":["t1","t2"],
            "operator":"and"
          }
        }
      ]
    }
  }
  ' http://xxxxxx/$db_name/$space_name/_search?size=10


批量查询(多个查询特征拼接在一起，比如一个特征使用10个float描述，批量查询20条，将20个10维特征按顺序拼接成feature即200个float的数组)
::
  curl -XPOST -H "content-type: application/json" -d'
  {
    "query":{
      "sum" : [
        {
           "field": "feature1",
           "feature": [0, 0, 0, 0, 0, 0],
        }
      ]
    }
  }
  ' http://xxxxxx/$db_name/$space_name/_msearch?size=10  


多向量查询(如下示例一条记录包含两个向量)
::
  curl -XPOST -H "content-type: application/json" -d'
  {
    "query":{
      "sum" : [
        {
           "field": "feature1",
           "feature": [0, 0, 0, 0, 0, 0],
        },
        {
           "field": "feature2",
           "feature": [0, 0, 0, 0, 0, 0]
        }
      ]
    }
  }
  ' http://xxxxxx/$db_name/$space_name/_search?size=10

