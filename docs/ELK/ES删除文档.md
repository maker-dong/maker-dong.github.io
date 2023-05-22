# ES删除文档

# 根据查询删除

使用`_delete_by_query`

查询条件详情见[ES查询语句-query](ELK/ES查询语句#query查询)

## 单查询条件删除

```json
POST {index_name}/_delete_by_query
{
  "query":{
    "term":{
      "_id":4043
    }
  }
}
```

## 多查询条件删除

```json
POST {index_name}/_delete_by_query
{
  "query": {
    "bool": { // bool查询
      "must": [], 
      "must_not": [], 
      "should": []
    }
  }
}
```

## 范围删除

```json
POST {index_name}/_delete_by_query
{
  "query": {
    "bool": {
      "must": [
        {
          "exists": {
            "field": "status"
          }
        },
        {
          "range": {
            "version_value": {
              "gte": 400,
              "lte": 405
            }
          }
        }
      ]
    }
  }
}
```

# 清空索引

## 清空单个索引

```json
POST {index_name}/_delete_by_query
{
  "query":{
    "match_all": {}
  }
}
```

## 清空多个索引

```json
POST {index1_name},{index2_name}/_delete_by_query
{
  "query":{
    "match_all": {}
  }
}
```

