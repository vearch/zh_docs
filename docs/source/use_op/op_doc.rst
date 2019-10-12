数据操作
=================


单条插入
--------

插入时指定唯一标识id, $doc_id
::

  curl -XPOST -H "content-type: application/json"  -d'
  {
    "area_code":"tpy",
    "product_code":"tpy",
    "image_type":"tpy",
    "image_name":"tpy",
    "image_vec": {
        "source":"test",
        "feature":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
    }
  }
  ' http://xxxxxx/$db_name/$space_name/$doc_id
 

不指定id使用url: http://xxxxxx/$db_name/$space_name 后台自动生成唯一标识id.


批量插入
--------

::

  curl -XPOST 
  http://xxxxxx/$db_name/$space_name/_bulk
 


更新
--------
更新时必须指定唯一标识id
::
   
  curl -XPOST -H "content-type: application/json"  -d'
  {
    "area_code":"tpy",
    "product_code":"tpy",
    "image_type":"tpy",
    "image_name":"tpy",
    "image_vec": {
        "source":"test",
        "feature":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
    }
  }
  ' http://xxxxxx/$db_name/$space_name/$doc_id


删除
--------
::

  curl -XDELETE http://xxxxxx/$db_name/$space_name/$doc_id

