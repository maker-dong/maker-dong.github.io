# ES采坑记录之maximum_shards_open

## 排查过程

其实之前我们也遇到过ES数据丢失的问题，一般来说是可能kafka或者数据源故障。因此我们先按照以往的经验排查了kafka，通过Prometheus监控，我们发现kafka的topic状态良好，每秒数据量正常，也没有出现积压的情况，所以排除了数据源与kafka的故障。

随后我们查看了Logstash的日志，问题浮出水面：

```
[WARN ][logstash.outputs.elasticsearch][main][d6cad0dd79cabf5273942b735be00f9dc4ccf8a0d605c856f3e7088b85385818] Could not index event to Elasticsearch. {:status=>400, :action=>["index", {:_id=>nil, :_index=>"XXX-log-2022.07.26", :routing=>nil, :_type=>"_doc"}, #<LogStash::Event:0x7a3115ec>], :response=>{"index"=>{"_index"=>"XXX-log-2022.07.26", "_type"=>"_doc", "_id"=>nil, "status"=>400, "error"=>{"type"=>"illegal_argument_exception", "reason"=>"Validation Failed: 1: this action would add [6] total shards, but this cluster currently has [9000]/[9000] maximum shards open;"}}}}
```

其中重要的信息在于：`this action would add [6] total shards, but this cluster currently has [9000]/[9000] maximum shards open`，意思就是说`这个操作会增加[6]个分片，但是这个集群目前最大的分片是[9000]/[9000]`。

因为我这个索引为3分区1副本，所以会创建6个分片，然而目前集群最大分片数为9000，已经达到上限，所以不能再添加分片了。怎么会有这种问题，猛男挠头。

## 解决问题

通过在上网搜索，原来Elasticsearch7.x版本之后，ES会默认给集群中每个节点设置1000个分片的上限，我们有9个节点，所以分片上限数为9000。好了，破案了。接下来我们要修改这个配置，加大分片上限数。

通过kibana的devtools执行：

```json
PUT /_cluster/settings
{
  "transient": {
    "cluster": {
      "max_shards_per_node":10000
    }
  }
}
```

或通过curl命令：

```shell
curl -XPUT 'http://xxx:9200/_cluster/settings' -H "Content-Type: application/json" -d '{"transient":{"cluster":{"max_shards_per_node":10000}}}'
```

将集群中每个节点的分片上限数改为10000，问题解决。



其实临时调整分片上限也只是缓兵之计，如果不对索引进行有效的管控，上限再大也有填满的一天。所以，做好ES索引的生命周期策略是十分重要的。