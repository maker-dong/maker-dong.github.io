# ES集群扩容

# 集群信息

ES版本：7.5.0

集群名称：es_cluster_7.5.0

| ip              | hostname | 角色        |
| --------------- | -------- | ----------- |
| 134.100.100.101 | node-1   | master/data |
| 134.100.100.102 | node-2   | master/data |
| 134.100.100.103 | node-3   | master/data |
| 134.100.100.104 | node-4   | data        |
| 134.100.100.201 | node-5   | 新增data    |
| 134.100.100.202 | node-6   | 新增data    |
| 134.100.100.203 | node-7   | 新增data    |

# 需求

水平扩容，新增3台节点作为ES集群的data节点。

# 过程

## 前期准备

与ES安装基本一致，此处简单叙述。

### 分发安装包

此处为与已有集群保持一致，选择使用二进制安装包进行安装。

将安装包分发至各机器并解压，但**先不要启动**。

### 切换es用户

es需要使用root以外的用户启动，此处我们新建一个esuser用户，并将es的安装目录权限的所有权更改为esuser。

```
chown -R esuser:esuser elasticsearch-7.5.0/
```

### 配置JDK

建议使用es安装包内自带的jdk，可以配置到es用户的`~/.bash_profile`中，不影响系统的环境变量。

```
JAVA_HOME=/data/elasticsearch-7.5.0/jdk
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME
export PATH
export CLASSPATH
```

## 修改配置文件

编辑`elasticsearch.yml`

### 设置集群名称

```
cluster.name: es_cluster_7.5.0
```

此处名称要**与已有集群保持一致**。

### 设置NodeName

```
node.name: node-5
```

此处要确保不能和已有node的名称重复。

### 设置不为主节点

```
node.master: false
```

此处我添加的是data节点，不让其参与选主，所以设置为false，如果可选主则设置为true。

### 设置节点发现列表

```
discovery.seed_hosts: ["134.100.100.101","134.100.100.102","134.100.100.103","134.100.100.104","134.100.100.201","134.100.100.202","134.100.100.203"]
```

将集群中的各个节点ip加入发现列表。

### 设置安全认证（可选）

如果原集群开启了xpack，则新加入的节点也应该开启相应的配置。

```
xpack.security.enabled: true
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
```

并且将原集群中某节点的证书拷贝至该新节点对应目录中：

```
scp node-1:/data/elasticsearch-7.5.0/config/certs/elastic-certificates.p12 /data/elasticsearch-7.5.0/config/certs
# 别忘了权限
chown -R esuser:esuser /data/elasticsearch-7.5.0/config/certs
```

### 启动新节点

```
nohup bin/elasticsearch 2>&1 &
```

观察日志，有问题解决问题，没问题则启动成功。

### 观察节点状态

```
curl -u elastic:'xxxxxx' -XGET 'http://node-1:9200/_cat/nodes'
192.100.100.102  46 95 2 1.62 1.60 1.59 dilm * node-2
134.100.100.201  4 73 1 0.29 0.24 0.25 dil  - node-5
134.100.100.103  65 78 0 0.17 0.15 0.16 dilm - node-3
134.100.100.101  46 82 1 0.67 0.64 0.56 dilm - node-1
134.100.100.104   7 94 1 0.32 0.31 0.32 dil  - node-4
```

可以看到新加入的node-5已经在集群中。

重复以上步骤将3台新节点都加入集群。

### 尝试连接新节点

在所有节点加入集群成功后，尝试连接新加入的节点查看集群状态。

```
curl -u elastic:'xxxxxx' -XGET 'http://node-5:9200/_cluster/health?pretty'
{
  "cluster_name" : "es_cluster_7.5.0",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 7,
  "number_of_data_nodes" : 7,
  "active_primary_shards" : 33,
  "active_shards" : 66,
  "relocating_shards" : 2,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 5,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 4879139,
  "active_shards_percent_as_number" : 100.0
}
```

可以看到新节点已经可以使用，集群中共7个节点，集群状态为green。因为有新节点加入，集群中有2个分片在进行迁移。