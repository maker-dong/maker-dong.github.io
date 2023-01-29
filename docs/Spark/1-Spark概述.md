# **Spark概述**

# **Spark是什么**

Spark是一个大规模数据处理的统一分析引擎

## **Spark的特点**

- 计算速度快

- - 基于内存
  - 优秀的作业调度策略

- 简洁易用

- - 支持Java、Scala、Python、R等多种语言
  - 提供多种数据处理操作，使数据处理更便捷

- 通用

- - Spark提供了多种组件，分别对应不同的计算需求

    ![image-20230129105135146](http://image.coolcode.fun/images/202301291051319.png)

- 多种运行模式

- - local——本地模式
  - standalone——独立集群模式
  - standalone-HA——高可用模式（生产环境使用）
  - on Yarn——Yarn集群模式（生产环境使用）
  - on Messos / on Cloud / on k8s ...

## **Spark发展历史**

- 2009年诞生于加州大学柏克莱AMPLab
- 2013年提交给Apache基金会；发布Spark Streaming、Spark Mllib、Shark
- 2014年成为Apache顶级项目；Spark1.0.0 发布；发布 Spark Graphx、Spark SQL代替Shark；
- 2017年发布里程碑版本Spark2.2.0，正式支持Structured Streaming

# **Spark基本原理**

## **名词解释**

- Application：指的是用户编写的Spark应用程序/代码，包含了Driver功能代码和分布在集群中多个节点上运行的Executor代码。
- Driver：Spark中的Driver即运行上述Application的Main()函数并且创建SparkContext，SparkContext负责和ClusterManager通信，进行资源的申请、任务的分配和监控等
- Cluster Manager：指的是在集群上获取资源的外部服务，Standalone模式下由Master负责，Yarn模式下ResourceManager负责;
- Executor：是运行在工作节点Worker上的进程，负责运行任务，并为应用程序存储数据，是执行分区计算任务的进程；
- RDD：Resilient Distributed Dataset弹性分布式数据集，是分布式内存的一个抽象概念；
- DAG：Directed Acyclic Graph有向无环图，反映RDD之间的依赖关系和执行流程；
- Job：作业，按照DAG执行就是一个作业；Job==DAG
- Stage：阶段，是作业的基本调度单位，同一个Stage中的Task可以并行执行，多个Task组成TaskSet任务集
- Task：任务，运行在Executor上的工作单元，一个 Task 计算一个分区，包括pipline上的一系列操作

## **Spark程序执行流程**

1. 当一个Spark Application被提交时，首先需要为这个Application构建基本的的运行环境，由任务控制节点Driver创建一个资源容器SparkContext
2. SparkContext向资源管理器注册并申请运行Executor资源
3. 资源管理器为Executor进程分配资源并启动Executor进程，Executor运行情况将随着心跳发送到资源管理器上
4. SparkContext根据RDD依赖关系构建DAG图，并提交给DAGScheduler进行解析，划分为Stage，并把Stage中的Task组成Taskset发送给TaskScheduler
5. Executor向SparkContext申请Task
6. TaskScheduler将Task发送给Executor运行，同时SparkContext将应用程序代码发送给Executor
7. Executor将Task丢入到线程池中执行，把执行结果返回给任务调度器，然后返回给DAG调度器，运行完毕后写入数据并释放资源

![image-20230129105201290](http://image.coolcode.fun/images/202301291052415.png)