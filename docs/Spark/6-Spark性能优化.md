# **Spark性能优化**

官方网址

http://spark.apache.org/docs/latest/configuration.html

http://spark.apache.org/docs/latest/tuning.html

# **基础优化**

## 资源参数

根据实际的任务数据量/优先级/公司集群的规模进行设置

## 并行度

原则是充分利用CPU的Core,一般可以设置为CPU核数的2~3倍

## 持久化+Checkpoint

## 使用广播变量

## 使用Kryo序列化

2.0之后对于简单基本类型及其集合类型+String类型及其集合类型默认使用的就是Kryo,自定义类型需要注册

```scala
public class MyKryoRegistrator implements KryoRegistrator{
	@Override
	public void registerClasses(Kryo kryo){
	  kryo.register(StartupReportLogs.class);
	}
}

//创建SparkConf对象
val conf = new SparkConf().setMaster(…).setAppName(…)
//使用Kryo序列化库
conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer");  
//在Kryo序列化库中注册自定义的类集合
conf.set("spark.kryo.registrator", "bigdata.com.MyKryoRegistrator");
```

## 数据本地化查找等待时间

```scala
val conf = new SparkConf().set("spark.locality.wait", "6")              
```

![image-20230129112135242](http://image.coolcode.fun/images/202301291121388.png)

# **RDD算子优化**

## 复用RDD

![image-20230129112246266](http://image.coolcode.fun/images/202301291122396.png)

## 尽早对数据进行过滤

对于大量的数据可以尽早进行filter过滤,减少后续操作的成本

## 读取大量小文件可以使用wholeTextFiles

```scala
object WordCount_bak2 {
 def main(args: Array[String]): Unit = {
   //1.创建SparkContext
   val conf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")//设置运行参数
   val sc: SparkContext = new SparkContext(conf)//创建sc
   sc.setLogLevel("WARN") //设置日志级别

   //2.读取文件夹
   //RDD[(文件名, 内容)]
   val filesRDD: RDD[(String, String)] = sc.wholeTextFiles("D:\\data\\spark\\files", minPartitions = 3)
   //_._2不再是之前的每一行数据,而是每一个文件中的内容
   //而每一个文件中的内容多行,所以用换行符进行切分
   //RDD[每一行]
   val linesRDD: RDD[String] = filesRDD.flatMap(_._2.split("\\r\\n"))
   //RDD[单词]
   val wordsRDD: RDD[String] = linesRDD.flatMap(_.split(" "))
   wordsRDD.map((_, 1)).reduceByKey(_ + _).collect().foreach(println)

   sc.stop()
  }
}
```

## 尽量使用mapPartition和foreachPartition

## 减少分区

可以使用`filter`过滤减少数据后,再使用`repartition/coalesce`减少分区

## 调节并行度

`repartition`/`coalesce`可以调节并行度

## 使用reduceByKey而不是groupByKey+sum

# shuffle

## **shuffle分类和bypass机制的开启**

1. 未经优化的HashShuffle:小文件过多

2. 优化之后的HashShuffle:小文件减少了,但是没有排序

3. 普通机制的sortShuffle:在形成中间小文件前进行排序,并且小文件带有索引编号

4. bypass机制的sortShuffle:小文件带有索引编号

当shuffle是reasTask数量小于200的时候开启bypass机制,大于200时使用普通机制的sortShuffle

在实际中,比如shufflet的readTask数量是300,默认会启动普通机制的sortShuffle会进行排序,但是可能后面的业务没有排序,那么shuffle时的排序就比较浪费性能,如何解决?

可以调整参数,将开启开启bypass机制的阈值调大为300以上

```scala
val conf = new SparkConf().set("spark.shuffle.sort.bypassMergeThreshold", "400")              
```

## map端和reduce端内存设置

```scala
val conf = new SparkConf().set("spark.shuffle.file.buffer", "64")              
```

上面参数调大可以减少map端溢写磁盘次数

```scala
val conf = new SparkConf().set("spark.reducer.maxSizeInFlight", "96")              
```

上面参数调大可以减少reduce端拉取次数

## **reduce端重试次数和等待时间间隔**

为了避免减少拉取次数,导致每次拉取数据时间间隔变长,导致任务拉取失败,可以调大拉取等待时间间隔和重试次数

```scala
val conf = new SparkConf().set("spark.shuffle.io.maxRetries", "6") val conf = new SparkConf().set("spark.shuffle.io.retryWait", "60s")              
```

# **数据倾斜**

## 自定义分区器-想怎么分区怎么分区

## 预处理导致倾斜的key

如加盐,hash

## 提高并行度



# **JVM参数调优**

## 降低cache内存提高用于计算的内存

```scala
val conf = new SparkConf().set("spark.storage.memoryFraction", "0.4")              
```

## 设置Executor堆外内存

```
--conf spark.yarn.executor.memoryOverhead=2048              
```

## 设置JVM进程间通信等待时间

```
--conf spark.core.connection.ack.wait.timeout=300              
```

