# ES集群滚动重启策略

## 前言

ES集群千万不要一次性重启全部节点，会导致所有分片被置为`unassigned_shards`，重启后需要大量时间重新分配切片。这期间集群会处于`red`状态，不能写入任何新数据。这在生产环境中会导致灾难性的后果。

综上，ES集群重启要采用轮流重启的方式。重启一个节点，等该节点重新加入并且集群状态变为green后，再重启下一个节点。而且**建议最后重启master节点**。

## 识别主节点的方法

```
GET /_cat/nodes
```

```
10.19.249.42 58 99 53 20.68 22.30 21.33 dilmrt - vplc06_es04
10.19.249.43 59 99 76 46.54 42.10 39.28 dilmrt - vplc07_es02
10.19.249.44 60 97 78 33.16 37.82 37.91 dilmrt - vplc08_es06
10.19.249.43 64 99 77 46.54 42.10 39.28 dilmrt - vplc07_es05
10.19.249.42 32 99 46 20.68 22.30 21.33 dilmrt - vplc06_es07
10.19.249.42 63 99 45 20.68 22.30 21.33 dilmrt - vplc06_es01
10.19.249.44 52 97 78 33.16 37.82 37.91 dilmrt - vplc08_es03
10.19.249.44 38 97 75 33.16 37.82 37.91 dilmrt - vplc08_es09
10.19.249.43 62 99 77 46.54 42.10 39.28 dilmrt * vplc07_es08
```

前面有`*`标注的就是主节点。

## 重启步骤

### 1. 停止数据写入

### 2. **临时关闭分片复制功能**

   根据ES的自平衡机制，当一个节点宕机时，集群会复制该节点上的分片到其他节点进行重新平衡，这会造成大量的网络与磁盘IO，所以我们暂时将其关闭。

   控制台：

   ```shell
   PUT /_cluster/settings
   {
     "transient": {
       "cluster.routing.rebalance.enable": "none"
     }
   }
   ```

   curl：

   ```shell
   curl -XPUT http://xxxxxx:9200/_cluster/settings -d '{
       "transient" : {
           "cluster.routing.allocation.enable" : "none"
       }
   }'
   ```

   执行命令后返回：

   ```json
   {
       "acknowledged": true,
       "persistent": {},
       "transient": {
           "cluster": {
               "routing": {
                   "allocation": {
                       "enable": "none"
                   }
               }
           }
       }
   }
   ```

### 3. 重启集群

   依次重启集群中的各个节点（建议主节点最后重启）。

   每次重启完一个节点后观察是否正常加入集群，正常加入集群后再执行下一个。

### 4. 开启分片复制机制

   当集群全部重启后恢复开启分片复制机制：

   控制台：

   ```shell
   PUT /_cluster/settings
   {
     "transient": {
       "cluster.routing.rebalance.enable": "all"
     }
   }
   ```

   curl:

   ```shell
   curl -XPUT http://xxxxxx:9200/_cluster/settings -d '{
       "transient" : {
           "cluster.routing.allocation.enable" : "all"
       }
   }'
   ```

   

## 加速重启后的分片恢复速度

增大这个设置项的值：`cluster.routing.allocation.node_initial_primaries_recoveries`
含义：一个节点上，同时处于初始化过程的主分片数量。默认为4

```shell
curl -XPUT "xxxx:9200/_cluster/settings" -H "Content-type: application/json" -d '
{
	"transient":
	{
		"cluster.routing.allocation.node_initial_primaries_recoveries":32
	}
}'
```

