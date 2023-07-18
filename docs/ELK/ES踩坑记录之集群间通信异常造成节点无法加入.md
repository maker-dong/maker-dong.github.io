# ES踩坑记录之集群间通信异常造成节点无法加入



# 问题描述

公司新搭了一套ES集群，4台机器，ES版本7.5.0，前期搭建十分顺利，但集群运行一段时间后会出现问题。问题具体体现为节点间通讯异常，集群会重新选主，但选主之后只能通过新的主节点进行集群操作，其他节点无法加入主节点。

通过查询ES的日志，我们发现如下报错：

```
[WARN ][o.e.c.s.MasterService    ] [node-1] failing [elected-as-master ([2] nodes joined)[{node-2}{lY51PsdiSW-kBOYQFYjQQw}{HtVGqYX2QRyEjVwQJQEVvA}{134.85.21.43}{134.85.21.43:9303}{dilmrt}{ml.machine_memory=67385552896, ml.max_open_jobs=20, xpack.installed=true, transform.node=true} elect leader, {node-1}{8hu8HMjLRJSJoDNtxtJ1LQ}{bT_S2fTXSeq8GfCwsHUsAA}{134.85.21.42}{134.85.21.42:9303}{dilmrt}{ml.machine_memory=67385552896, xpack.installed=true, transform.node=true, ml.max_open_jobs=20} elect leader, _BECOME_MASTER_TASK_, _FINISH_ELECTION_]]: failed to commit cluster state version [986]
org.elasticsearch.cluster.coordination.FailedToCommitClusterStateException: publication failed
 at org.elasticsearch.cluster.coordination.Coordinator$CoordinatorPublication$4.onFailure(Coordinator.java:1467) ~[elasticsearch-7.9.0.jar:7.9.0]
 at org.elasticsearch.action.ActionRunnable.onFailure(ActionRunnable.java:88) ~[elasticsearch-7.9.0.jar:7.9.0]
 at org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:39) ~[elasticsearch-7.9.0.jar:7.9.0]
 at org.elasticsearch.common.util.concurrent.EsExecutors$DirectExecutorService.execute(EsExecutors.java:226) ~[elasticsearch-7.9.0.jar:7.9.0]
 at org.elasticsearch.common.util.concurrent.ListenableFuture.notifyListener(ListenableFuture.java:106) ~[elasticsearch-7.9.0.jar:7.9.0]
 at org.elasticsearch.common.util.concurrent.ListenableFuture.addListener(ListenableFuture.java:68) ~[elasticsearch-7.9.0.jar:7.9.0]
 at org.elasticsearch.cluster.coordination.Coordinator$CoordinatorPublication.onCompletion(Coordinator.java:1390) ~[elasticsearch-7.9.0.jar:7.9.0]
 at org.elasticsearch.cluster.coordination.Publication.onPossibleCompletion(Publication.java:125) ~[elasticsearch-7.9.0.jar:7.9.0]
 at org.elasticsearch.cluster.coordination.Publication.onPossibleCommitFailure(Publication.java:173) ~[elasticsearch-7.9.0.jar:7.9.0]
 at org.elasticsearch.cluster.coordination.Publication.access$500(Publication.java:42) ~[elasticsearch-7.9.0.jar:7.9.0]
```

大体意思就是说node-1无法加入node-2为主节点的集群。

这里有两个问题，首先node-1原本是主节点，为什么它要加入别人的节点？其次，为什么node-1无法加入集群？

# 问题分析

从现象上看，集群中原本node-1为主节点，现在node-2成了主节点，也就是说原本主节点出现了问题，导致集群重新选主。但通过对日志的观察，我们没有看到集群有明显的错误。

通过在网络上的搜到的解决办法，可以调整ES集群的连接超时时间配置

```
cluster.publish.timeout: 15s
cluster.fault_detection.leader_check.timeout: 5s
cluster.fault_detection.follower_check.timeout: 5s
cluster.follower_lag.timeout: 10s
```

修改之后还是没有效果，集群正常运行一段时间后还是会发生异常。

经过一阵研究我们发现，原来还是主机之间的通讯有些问题，我们需要修改一下主机之间的通讯保持参数。`/etc/sysctl.conf`:

```
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3 
net.ipv4.tcp_keepalive_intvl = 60 
```

执行命令试配置生效：

```
sysctl -p
```

