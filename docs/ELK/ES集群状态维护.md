# ES集群状态维护


# ES状态查询

- 查看集群健康

  ```
  GET /_cluster/health
  ```

- 查看索引健康

  ```
  GET /_cluster/health?pretty&level=indices
  ```

- 查看分片监健康

  ```
  GET /_cluster/health?pretty&level=shards
  ```

- 查看集群数据统计

  ```
  GET /_cluster/stats
  ```

- 查看节点信息

  ```
  GET /_cat/nodes?v=true
  ```

  - heap.percent：配置的最大堆内存
  - ram.percent：已用总内存百分比
  - load_1m：最近的平均负载
  - load_5m：最近5分钟内的平均负载
  - load_15m：最近15分钟内的平均负载
    - node.role：节点的角色
    - c（冷节点）
    - d（数据节点）
    - f（冻结节点）
    - h（热节点）
    - i（摄取节点）
    - l（机器学习节点）
    - m（主节点）
    - r（远程集群客户端节点）
    - s（内容节点）
    - t（转换节点）
    - v（仅投票节点）
    - w（热节点）
    -  -（仅协调节点）

- 查看索引信息

  ```
  GET /_cat/indices/xxx*
  ```

# 集群维护

可参考[ES集群滚动重启策略](ES集群滚动重启策略)

- 停止分片自平衡

  ```
  PUT /_cluster/settings
  {
    "transient": {
      "cluster.routing.rebalance.enable": "none"
    }
  }
  ```

- 开启分配自平衡

  ```
  PUT /_cluster/settings
  {
    "transient": {
      "cluster.routing.rebalance.enable": "all"
    }
  }
  ```

- 查看集群设置

  ```
  GET /_cluster/settings
  ```

- 查看恢复情况

  ```
  GET /_recovery?pretty
  ```

# 集群健康指标解析

`/_cluster/health`是最常用的ES集群健康度查询的指令，返回如下：

```json
{
  "cluster_name" : "elasticsearch_vplc",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 9,
  "number_of_data_nodes" : 9,
  "active_primary_shards" : 4504,
  "active_shards" : 8728,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

- `status`：es集群状态

  - **green**：所有的主分片和副本父分片都已分配，集群100%可用。
  - **yellow**：所有的主分片已经分片，但至少还有一个副本分片是缺失的。不会有数据丢失，所以搜索结果依然是完整的。不过高可用性在某种程度上被弱化。如果更多的分片消失，就会丢数据。
  - **red**：至少一个主分片（以及它的全部副本）都在丢失中。搜索只能返回部分数据，而分配到这个分片的写入请求会返回一个异常。

- `timed_out`：是否超时

- `number_of_nodes` ：节点数

- `number_of_data_nodes`：数据节点数

- `active_primary_shards`：集群中所有索引的主分配数

- `active_shards`：集群中所有索引的分片数

- `relocating_shards`：当前正在节点间迁移的分片，该值在正常情况下为0，但在ES集群有节点加入或移除时，集群会发现分片分布不均衡，便会开始进行分片迁移。

- `initializing_shards`：初始化中的分片数。往往在分片刚被创建或节点重启时，会经历短暂的`initializing`状态。

- `unassigned_shards`：未分配的分片数。表示集群中存在分片但实际又找不到分片，常见于存在未分配的副本，如集群中有1个节点，有一个索引有5个分片1个副本，由于ES的灾备原则，副本分片不能与主分配保存在同一个节点中，那么就会有5个副本分片处于未分配状态。

  > 如果你的集群是 `red` 状态，也会长期保有未分配分片（因为缺少主分片）。