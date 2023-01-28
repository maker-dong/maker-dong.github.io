# ES远程集群设置与使用

> Elasticsearch在5.3版本中引入了Cross Cluster Search（CCS 跨集群搜索）功能，用来替换掉要被废弃的Tribe Node。类似Tribe Node，Cross Cluster Search用来实现跨集群的数据搜索。**跨集群搜索**使您可以针对一个或多个[远程集群](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-remote-clusters.html)运行单个搜索请求 。例如，您可以使用跨集群搜索来过滤和分析存储在不同数据中心的集群中的日志数据。

# 1 配置远程集群

ES提供两种远程集群配置方案，一种是直接在ES的配置文件`elasticsearch.yml`中进行配置，另一种是使用API的方式进行设置。

## 1.1 配置文件配置方式

修改`elasticsearch.yml`，添加如下内容

```yml
search:
    remote:
        cluster01:
            seeds: 192.168.10.101:9300
            seeds: 192.168.10.102:9300
            seeds: 192.168.10.103:9300
            transport.compress: true 
            skip_unavailable: true 
```

- `cluster01`：集群名称，自定义即可
- `seeds`：集群的节点列表，可以配置一个或多个
- `transport.ping_schedule`:  使用ping检测连接状态的时间间隔
- `skip_unavailable`：跨集群搜索是否跳过不可用集群

## 1.2 API配置方式

 使用`Cluster Settings API`进行设置

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster01": {
          "skip_unavailable": false,
          "mode": "sniff",
          "proxy_address": null,
          "proxy_socket_connections": null,
          "server_name": null,
          "seeds": [
            "cluster01_node01:9300",
            "cluster01_node02:9300",
            "cluster01_node03:9300"
          ],
          "node_connections": 3
        }
      }
    }
  }
}
```

配置参数同上

在此我更倾向于使用API的方式设置远程集群，这样更方便对远程集群进行修改。

## 1.3 查看远程集群状态

使用`GET _remote/info`请求进行查看

```json
{
  "cluster01" : {
    "connected" : true,
    "mode" : "sniff",
    "seeds" : [
        "192.168.10.101:9300",
        "192.168.10.102:9300",
        "192.168.10.103:9300"
    ],
    "num_nodes_connected" : 3,
    "max_connections_per_cluster" : 3,
    "initial_connect_timeout" : "30s",
    "skip_unavailable" : false
  }
}

```

## 1.4 删除远程集群

如果设置有误或不想用了，可以将远程集群删除，其实就是将seeds设置为空。

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster01": {
          "seeds": null 
        }
      }
    }
  }
}
```

## 1.5 使用kibana管理远程集群

如果使用了kibana，那么设置远程集群更加简单，只需要在页面上操作即可。

打开`Stack Management`，左侧目录找到`远程集群`或`remote cluster`，在打开的页面点击`添加远程集群`

填写集群名称与节点信息

![image-20211029163006356](http://image.coolcode.fun/images/202110291630432.png)

保存即可

# 2 使用远程集群搜索

## 远程集群权限设置

1. 在远程集群上创建与本地集群同名的角色
2. 远程集群上的角色要赋予对应索引的`read`与`read_cross_cluster`权限，否则本地集群访问该索引时连接会被拒绝。

## 查询远程集群

查询远程集群的索引需要指定集群名称

```
GET /cluster_name:index/_search
```

同时查询多个集群

```
GET /cluster_name:index,cluster_name:index/_search
```

同时查询所有集群

```
GET */index/_search
```

## Kibana中创建索引模式

在Kibana中创建索引模式时，也要指定集群名

```
cluster_name:index*
```

之后使用就跟本地集群的索引模式一样了。

