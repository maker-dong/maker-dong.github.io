# ES更改索引字段数据类型

## 创建新索引并指定字段映射

此处我们要将callTimeLength字段映射为long类型

```
PUT /index-2
{
  "mappings": {
      "properties": {
        "callTimeLength": {
          "type": "long"
        }
      }
    }
}
```



## 将旧索引数据reindex到新索引

```
POST _reindex
{
  "source": {
    "index": "index-1"
  },
  "dest": {
    "index": "index-2"
  }
}
```

关于reindex的介绍放在最后[reindex介绍](# 附：reindex介绍)

执行成功后要等待一段时间，根据要reindex数据的大小需要等待不同的时间完成数据的复制。



## 将旧索引新建并reindex

等待上一步reindex完成后，index-2中的数据就与index-1中的数据相同了。

**接下来删除index-1**

```
DELETE /index-1
```

**然后新建index-1**

```
PUT /index-1
{
  "mappings": {
      "properties": {
        "callTimeLength": {
          "type": "long"
        }
      }
    }
}
```

**再将index-2中的数据reindex到新的index-1**

```
POST _reindex
{
  "source": {
    "index": "index-2"
  },
  "dest": {
    "index": "index-1"
  }
}
```

等待数据reindex完成后，查看index-1的映射，数据完成了callTimeLength字段的数据类型更改。



## 附：reindex介绍

### 基本介绍

> The most basic form of `_reindex` just copies documents from one index to another. 
>
> （_reindex 最基本的形式只是将文档从一个索引复制到另一个索引。）

### 适用场景

1. 分片数变更：当你的数据量过大，而你的索引最初创建的分片数量不足，导致数据入库较慢的情况，此时需要扩大分片的数量，此时可以尝试使用Reindex。
2. mapping字段变更：当数据的mapping需要修改，但是大量的数据已经导入到索引中了，重新导入数据到新的索引太耗时；但是在ES中，一个字段的mapping在定义并且导入数据之后是不能再修改的，所以这种情况下也可以考虑尝试使用Reindex。
3. 分词规则修改，比如使用了新的分词器或者对分词器自定义词库进行了扩展，而之前保存的数据都是按照旧的分词规则保存的，这时候必须进行索引重建。

### 基本用法

```
POST _reindex
{
  "source": {
    "index": "index-old"
  },
  "dest": {
    "index": "index-new"
  }
}
```

[ES官方reindex文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/docs-reindex.html)

