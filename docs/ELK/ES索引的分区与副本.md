# ES索引的分区与副本

# 概念

分区与副本都是ES索引的重要机制，那么二者具体有何不同呢？

- 分区（shard）

  ES是一个分布式系统，ES中的索引可以被分解为多个较小的分片，将这些分片分配到不同的节点上。当查询该索引时，ES会将查询发送给每个相关的分片，再将查询结果进行合并。分区的目的是为了避免单分区时数据量过大，对节点的CPU、内存、磁盘产生压力过大，同时增加了集群的可扩展性。

- 副本（replia）

  副本即副本分片，顾名思义是对数据进行分片数据的复制。ES中允许存在多个数据相同的分片，把进行更改操作的分片称为主分配，其余分片均为副本分片。当主分配数据不可用时，副本分片会自动成为新的主分配，以保障数据的高可用性。

# 关系

通过上述的解释我们可以知道，分区是将数据切割，副本是将数据进行备份，那么二者的关系是什么呢？

此处我们举个栗子：

假设我们为某个索引设置分区数为3，副本数为1，那么该索引的主分片数为3，副本分片数为3，共6个分片。

也就是说，每个主分配都会有自己对应的副本分片。

![ES分片](http://image.coolcode.fun/images/202207121547049.png)

ES会将主分配分配到集群中不同机器上面，副本分片不会与自己对应的主分片分配到同一台机器，因为如果该机器出现故障时，主分配与副本分片同时丢失，无法起到容错的作用。

# 合理分配

- 分片分配过大会导致单个分片数据过多，服务器压力过大，分片分配过小则会产生更多的分片，造成小的分段，增加开销。通常分片大小设置为20-40G之间是比较合理的。

- 集群中每个节点保存的分片数量与节点的堆内存大小是成正相关的，目前比较好的经验是每个节点的分片数量低于每GB堆内存配置20到25个分片。由于ES推荐的最大JVM堆空间为32G，若ES节点堆内存大小设置为32G，则单个节点的分片数量维持在640-800个节点比较科学。

- 如果担心数据增长过快，若节点的JVM的堆内存大小设置为最大推荐值32G，则单个分片的最大容量控制在30G以内是比较合理的。
- 如果从节点的角度去考虑，单个索引的副本数建议最大不要超过节点数量的3倍。

# 设置方法

## 新建索引时设置

```shell
PUT /newindex
{
   "settings" : {
      "number_of_shards" : 3, //分区数
      "number_of_replicas" : 1 //副本数
   }
}
```

## 修改已存在的索引

**注意：已经创建的索引只能修改副本数，不能修改分区数**

```shell
PUT /newindex/_settings
{
   "number_of_replicas" : 2
}
```

## 使用索引模板

在索引模板中配置索引设置

```shell
{
  "index": {
    "number_of_shards": "3"
  }
}
```

