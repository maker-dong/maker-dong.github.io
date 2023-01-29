# **Spark中的RDD**

# **概述**

RDD是Spark中的弹性分布式数据集，其特点为不可变、可分区、元素可并行计算。

## **优势**

RDD的优势体现在：

- 解决了运行效率问题——中间结果保存在内存中，运行效率高
- 解决了开发效率问题——RDD支持丰富的API
- 封装了复杂的分布式操作——容错、调度、位置感知等

## **五大属性**

- 分区列表
- 计算函数
- 依赖关系
- 分区函数
- 最佳位置

# **RDD的API**

## **RDD的创建**

- 从文件系统创建

  ```
  val rdd = sc.textFile(path)
  ```

- 从已有RDD转换

  ```
  val rdd2 = rdd1.flatmap(_.split(" "))    
  ```

- 通过集合创建

  ```
  val rdd3 = sc.parallelize(Array(1,2,3,4,5,6,7,8)) 
  val rdd4 = sc.makeRDD(List(1,2,3,4,5,6,7,8))
  ```

## **RDD的方法/算子**

- Transformation 转换操作

  返回一个新的RDD

- Action 动作操作

  返回值不是RDD（无返回值或返回其他类型）

![image-20230129105525211](http://image.coolcode.fun/images/202301291055318.png)

**注意：转换操作不会立即执行，需要执行动作操作来触发**

## **RDD分区**

### **目的**

提高计算的并行度，提高计算效率

### **原则**

是分区个数尽量等于集群中CPU的核心数。

注意：实际开发中往往将并行度设置为CPU核心数的2~3倍，以充分压榨CPU的计算资源。

### **用法**

repartition:重分区.可以增加或减少分区,注意:原来的rdd分区数不变,改变的是新生成的rdd的分区数

## **其他常用API**

- map : 对RDD中的每一个元素进行操作,并返回操作的结果
- filter: 对RDD中的每一个元素进行过滤,留下返回True的
- flatMap: 对RDD中的每一个元素进行操作,并压扁,再返回
- sortBy:根据指定的规则进行排序,true表示升序
- 交集:intersection ,并集:union, 差集:subtract,笛卡尔积:cartesian
- jion:根据key进行jion, leftOutJoin:左外连接
- groupBykey:根据key进行分组
- groupBy:根据指定的规则进行分组
- reduce:聚合
- reduceByKey:根据key进行聚合
- collect:收集数据到本地,注意数据量大的时候慎用
- count:求RDD最外层元素的数量
- distinct:去重
- top:去排好序的前n个
- take:取原来顺序的前n个
- keys/values:取所有的key/取所有的value
- mapValues:对所有的value进行操作,key不变
- foreach和foreachPartition:
- map和mapPartition

# **RDD缓存/持久化**

## **目的**

在开发中对于频繁使用的RDD可以将其进行缓存/持久化，再次使用时可以直接读取，提高计算效率。

## **操作**

对要持久化的RDD进行persist操作，并设置存储级别。

```
rdd.persist(StorageLevel.MEMORY_AND_DISK)
```

## **存储级别**

| 持久化级别                           | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| MEMORY_ONLY(默认)                    | 将RDD以非序列化的Java对象存储在JVM中。 如果没有足够的内存存储RDD，则某些分区将不会被缓存，每次需要时都会重新计算。 这是默认级别。 |
| MEMORY_AND_DISK(开发中可以使用这个)  | 将RDD以非序列化的Java对象存储在JVM中。如果数据在内存中放不下，则溢写到磁盘上．需要时则会从磁盘上读取 |
| MEMORY_ONLY_SER (Java and Scala)     | 将RDD以序列化的Java对象(每个分区一个字节数组)的方式存储．这通常比非序列化对象(deserialized objects)更具空间效率，特别是在使用快速序列化的情况下，但是这种方式读取数据会消耗更多的CPU。 |
| MEMORY_AND_DISK_SER (Java and Scala) | 与MEMORY_ONLY_SER类似，但如果数据在内存中放不下，则溢写到磁盘上，而不是每次需要重新计算它们。 |
| DISK_ONLY                            | 将RDD分区存储在磁盘上。                                      |
| MEMORY_ONLY_2, MEMORY_AND_DISK_2等   | 与上面的储存级别相同，只不过将持久化数据存为两份，备份每个分区存储在两个集群节点上。 |
| OFF_HEAP(实验中)                     | 与MEMORY_ONLY_SER类似，但将数据存储在堆外内存中。 (即不是直接存储在JVM内存中)如：Tachyon-分布式内存存储系统、Alluxio - Open Source Memory Speed Virtual Distributed Storage |

# **RDD的容错机制checkpoint**

## **目的**

缓存/持久化并不能保证RDD的绝对安全，使用checkpoint将频繁使用且重新计算代价较高的RDD放在HDFS上，以保证安全。

## **操作**

先设置checkpoint在HDFS上的目录，再使用要存储的RDD调用checkpoint方法。

```
sc.setCheckpointDir("hdfs目录") rdd.checkpoint
```

**注意：开发中一般对频繁使用且重新计算代价较高的RDD要先进行缓存/持久化在，再进行Checkpoint。**

# **RDD的依赖关系**

## **宽依赖**

父RDD的一个分区被子RDD的多个分区所依赖。

宽依赖是划分stage的依据，必须等上一阶段完成才能进行下一阶段。

![image-20230129105720191](http://image.coolcode.fun/images/202301291057312.png)

## **窄依赖**

父RDD的一个分区只被一个子RDD分区所依赖。

不同分区可并行计算。

窄依赖的一个分区中数据如果丢失，只需要重新计算对应分区的数据即可重新获取。

![image-20230129105732324](http://image.coolcode.fun/images/202301291057430.png)

# **DAG的生成与Stage的划分**

## **DAG**

DAG（有向无环图）指的是数据转换执行过程，有方向，无闭环。

开始创建：通过SparkContext创建RDD

结束创建：触发Action，形成完整的DAG

## **Stage**

Stage（阶段）是DAG中数据转换的阶段。

## **划分Stage的依据**

将DAG从后往前依次判断，遇到宽依赖就划分一个Stage，遇到窄依赖就加入到当前Stage中。

![image-20230129105806574](http://image.coolcode.fun/images/202301291058708.png)

一个Spark程序中可以有多个DAG

一个DAG可以有多个Stage

一个Stage可以有多个Task并行执行（Task数=分区数）

# **RDD的数据源**

- 文本文件
  - HadoopAPI：
    - textFile底层使用HadoopFile实现 
    - saveAsTextFile底层使用saveAsHadoopFile 
    - SequenceFile文件 对象文件 

```
sc.textFile("./dir/*.txt") 
//读 
sc.objectFile[k,v](path) //因为是序列化所以要指定类型 
//写 
RDD.saveAsObjectFile() 
```

- 数据库

- - HBase
  - JDBC

# **Spark共享变量**

在spark程序中，当一个传递给Spark的函数（例如map和reduce）在集群点上面运行时，Spark操作实际上操作的是这个函数所用变量的一个副本。这些变量会被复制到每台机器上，并且这些变量在远程机器上的所有更新都不会传递回驱动程序。

通常来说任务间的变量读写效率低，因此Spar提供了两种共享变量：广播变量（broadcast variable）和累加器（accumulator）。

广播变量用来高效分发较大的对象，累加器用来对信息进行聚合。

## **广播变量**

### **目的**

在Spark中，若Driver要向各个Task分发对象，Spark默认会将对象推送给各个Task。如果Task数目很多，Driver的带宽会成为系统的瓶颈，并且会大量消耗Task所在服务器的资源。

使用广播变量的方式，将对象分发到每个Executor，Executor上的Task会共享这个变量，需要时从Executor拉取变量。

### **操作**

设置广播变量

```
//创建broadcast变量，并将list的值存入 
val broadCast = sparkcontext.broadcast(list)              
```

使用广播变量

```
//取出广播变量中的值 
broadCast.value              
```

 **变量只会被发到各个节点一次，并作为只读值处理，修改这个值不会影响到别的节点。**

广播变量只能在Driver端定义，不能在Executor端定义。

## **累加器**

### **目的**

将工作节点中的值聚合到驱动器程序中。

### **操作**

设置累加器

```
//创建accumulator并初始化为0 
val accumulator = sc.accumulator(0);              
```

使用累加器

```
val result = linesRDD.map(s => {
	accumulator.add(1) //有一条数据就增加1   
	s 
}) //取出累加器中的值 accumulator.value              
```

**注意：累加器在Driver端定义赋初始值，累加器只能在Driver端读取，在Excutor端更新。**