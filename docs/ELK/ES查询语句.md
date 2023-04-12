# 基本查询语法

## 查询语法结构

```
GET /索引名/_search
{
	"from" : 0,  // 返回搜索结果的开始位置
	"size" : 10, // 分页大小，一次返回多少数据
	"_source" :[ ...需要返回的字段数组... ],
	"query" : { ...query子句... },
	"aggs" : { ..aggs子句..  },
	"sort" : { ..sort子句..  }
}
```

- `query`子句：query子句主要用来编写类似SQL的Where语句，支持布尔查询（and/or）、IN、全文搜索、模糊匹配、范围查询（大于小于）。

- `aggs`子句：aggs子句，主要用来编写统计分析语句，类似SQL的group by语句

- `sort`子句：sort子句，用来设置排序条件，类似SQL的order by语句

- `source`：_source用于设置查询结果返回什么字段，类似Select语句后面指定字段。

  例子:

	```
	GET /order_v2/_search
	{
	  "_source": ["order_no","shop_id"], 
	  "query": {
	    "match_all": {}
	  }
	}
	```

## 查询多个索引

```
GET /order1,order2/_search
GET /order*/_search
```

## 分页查询

ES查询的分页主要通过from和size参数设置，类似MYSQL 的limit和offset语句

例子：

```
GET /order_v2/_search
{
  "from": 0,
  "size": 20, 
  "query": {
    "match_all": {}
  }
}
```



# query查询

## match-匹配单个字段

```
GET /索引名/_search
{
  "query": {
    "match": {
      "FIELD": "TEXT"
    }
  }
}
```

说明：

- `match`实现全文搜索

- FIELD - 就是我们需要匹配的字段名
  - 如果字段数据类型是text，则搜索关键词会进行分词

- TEXT - 就是我们需要匹配的内容

## term-精确匹配（不分词）

```
GET /索引名/_search
{
  "query": {
    "term": {
      "FIELD": "VALUE"
    }
  }
}
```

说明：

- `term`类似于SQL语句中的`=`

- FIELD - 就是我们需要匹配的字段名

- VALUE - 就是我们需要匹配的内容，除了TEXT类型字段以外的任意类型。

## terms-属于

```
GET /索引名/_search
{
  "query": {
    "terms": {
      "FIELD": [
        "VALUE1",
        "VALUE2"
      ]
    }
  }
}
```

说明：

- `terms`类似与SQL语句中的`in`

- FIELD - 就是我们需要匹配的字段名

- VALUE1, VALUE2 … VALUE N - 就是我们需要匹配的内容，除了TEXT类型字段以外的任意类型。

## range-范围

```
GET /索引名/_search
{
  "query": {
    "range": {
      "FIELD": {
        "gte": VALUE1, 
        "lte": VALUE2
      }
    }
  }
}
```

说明：

- `range`实现范围查询

- FIELD - 字段名
- `gte`范围参数 - 等价于>=
- `lte`范围参数 - 等价于 <=
- 范围参数可以只写一个，例如：仅保留 “gte”: 10， 则代表 FIELD字段 >= 10
  范围参数如下：

范围参数：

- `gt` - ` > `
- `gte` -  `>=`
- `lt` -  `<`
- `lte` - `<=`

## 前缀查询

```
GET /索引名/_search
{
    "query": {
        "prefix": {
            "FIELD": "VALUE"
        }
    }
}
```

说明：

- `prefix`：实现前缀查询
- `FIELD`：要匹配的字段
- `VALUE`：需要匹配的内容

## 通配符查询

```
GET /索引名/_search
{
    "query": {
        "wildcard": {
            "FIELD": "*abc"
        }
    }
}
```

- `wildcard`：实现通配符查询
- `FIELD`：要匹配的字段
- `VALUE`：需要匹配的内容（需要包含通配符）

## bool组合查询

ES通过bool查询实现多条件的组合。

### bool基本查询语法

```
GET /索引名/_search
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

条件参数：

- `must`：必须匹配，类似于SQL中的`and`
- `must_not`：必须不匹配，类似于SQL中的`not`
- `should`：匹配其中一个即可，类似于SQL中的`or`

### must条件

```
GET /索引名/_search
{
  "query": {
    "bool": {
      "must": [
         {匹配条件},
         {匹配条件···}
        ]
    }
  }
}

```

说明：其中的匹配条件可以使用上面提到的`match`/`term`/`terms`等查询

### must_not条件

```
GET /索引名/_search
{
  "query": {
    "bool": {
      "must_not": [
         {匹配条件},
         {匹配条件···}
        ]
    }
  }
}
```

说明：其中的匹配条件可以使用上面提到的`match`/`term`/`terms`等查询

### should条件

```
GET /索引名/_search
{
  "query": {
    "bool": {
      "should": [
         {匹配条件},
         {匹配条件···}
        ]
    }
  }
}
```

