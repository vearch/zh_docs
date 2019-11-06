查询
==================


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



