# **SparkStreaming**

# **概述**

## **是什么**

SparkStreaming是Spark中的实时处理模块，用来对实时数据进行处理。

对于流中的实时数据，逐条处理对系统以及机器要求过高，SparkStreaming可以将实时到来的数据划分为一个个小批次，然后对每一个小批次的数据进行处理。

## **特点**

- 易用

- - 可以像编写离线批处理一样去编写流式程序
  - 支持Java/Scala/Python语言

- 容错

- - SparkStreaming自带容错机制，可以恢复丢失的工作

- 易整合到Spark

- - 流式处理与批处理和交互式查询相结合

## **应用场景**

SparkStreaming可以接收多种数据源发送的数据流，然后分批次处理，再将处理好的数据写入到文件系统或数据库中。

![image-20230129111250979](http://image.coolcode.fun/images/202301291112106.png)

# **SparkStreaming工作原理**

## **SparkStreaming数据抽象**

SparkStreaming的数据抽象是`DStream`，是一个**离散化的数据流**。

DStream的特点:

- DStream本质上就是一系列时间上连续的RDD
- 对DStream的操作就是对RDD的操作
- RDD的容错性决定了DStream的容错性
- DStream只能做到近实时的数据处理（0.5s~10s)

## **SparkStreaming工作流程**

数据源以Kafka为例

![image-20230129111325998](http://image.coolcode.fun/images/202301291113119.png)

# **DStream-API**

与RDD一样，DStream提供两个算子，Transformation与Action(Output)。

## **Transformation**

与RDD类似，需要单独说明的有：

### **transform(func)**

作用于DStream中的每个RDD，对每个RDD进行RDD-to-RDD的操作，返回新的RDD并组成新的DStream。

### **updateStateByKey状态更新**

对于批处理，每个批次的计算结果都是相对独立的，不能进行累加。如果需要累加，则需要使用updateStateByKey()操作来更新状态。

updateStateByKey(fun)方法中传入的参数为需要用来状态的方法，需要自定义。

代码示例：

```scala
///使用updateStateByKey需要先设置一个checkpoint目录,因为我们要对历史值进行聚合,历史值存储需要给它设置一个目录
streamingContext.checkpoint("./ssc") //实际开发写HDFS

//对所有数据进行聚合,使用updateStateByKey
val result: DStream[(String, Int)] = wordAndOneDStream.updateStateByKey(updateFunc)


/**
   * 主方法外定义一个updateFunc方法用来聚合所有的数据(将历史值+当前这一批次的值)
   * @param currentValues 当前这一批次的值[1,1,1,.....]
   * @param historyValue 历史值,第一次没有为None,我们应该把它设置为0,所以应该用getOrElse(0)
   * @return
   */
def updateFunc(currentValues:Seq[Int], historyValue:Option[Int]):Option[Int]={
   val result: Int = historyValue.getOrElse(0) + currentValues.sum
   Some(result)
}
```

### **reduceByKeyAndWindow窗口操作**

SparkStreaming提供了窗口操作，允许在数据转换中定义一个滑动窗口。

切分批次表示数据切分每个批次的时长。

窗口长度表示每次处理长时间的数据。

滑动间隔表示每隔多久窗口向前滑动一次。

对于窗口划分与滑动有如下三种方式：

![image-20230129111433795](http://image.coolcode.fun/images/202301291114946.png)

如图所示，当滑动间隔大于窗口长度时，数据会丢失，所有这种方式一般不会使用，开发中一般设置滑动间隔小于窗口长度或滑动间隔等于窗口长度。

代码示例：

```scala
//Seconds(5)表示每隔5秒对数据进行批次的划分 即切分批次
val ssc = new StreamingContext(sc, Seconds(5))
ssc.checkpoint("./ssc")

//使用ssc从Socket接收数据
val dataDStream: ReceiverInputDStream[String] = ssc.socketTextStream("node01",9999)

//对数据进行处理
val wordDStream: DStream[String] = dataDStream.flatMap(_.split(" "))
val wordAndOneDStream: DStream[(String, Int)] = wordDStream.map((_, 1))

//使用窗口操作,统计最近10s的数据,每隔5s统计一次
// @param windowDuration 窗口的长度 10s
// @param slideDuration 滑动间隔   5s
//windowDuration>slideDuration 部分数据会被重复(重新)计算,实际中用的最多
val result: DStream[(String, Int)] = wordAndOneDStream.reduceByKeyAndWindow((a:Int,b:Int)=>a+b,Seconds(10),Seconds(5))
```

## **Action/Output**

与RDD的类似，需要单独注意的有：

### **print**

直接打印DStream中的内容到控制台。

### **foreachRDD(func)**

对DStream中的每个RDD

# **案例：模拟热词排行**

```scala
import org.apache.spark.SparkContext
import org.apache.spark.sql.SparkSession
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
 * Desc 模拟热词排行，统计最近10s数据，每5s统计一次，取Top3
 */
object WordTopN {
 def main(args: Array[String]): Unit = {
   val spark: SparkSession = SparkSession.builder().appName("WordTopN").master("local[*]").getOrCreate()
   val sc: SparkContext = spark.sparkContext
   sc.setLogLevel("WARN")
   //每个5s对数据进行批处理
   val ssc = new StreamingContext(sc, Seconds(5))
   //历史值存放位置
   ssc.checkpoint("./ssc")
   //使用ssc从Socket接收数据
   val dataDStream: ReceiverInputDStream[String] = ssc.socketTextStream("node01", 9999)
   //数据处理 将每个单词按" "划分，并记为1
   val wordOneDS: DStream[(String, Int)] = dataDStream.flatMap(_.split(" ")).map((_, 1))
   //统计最近5s，每10s统计一次
   // windowDuration:10s
   // slideDuration: 5s
   val wordCountDS: DStream[(String, Int)] = wordOneDS.reduceByKeyAndWindow((a:Int, b:Int) => a + b, Seconds(10L), Seconds(5L))
   
   val result: DStream[(String, Int)] = wordCountDS.transform(rdd => {
     //求top3
     val sortArr: Array[(String, Int)] = rdd.sortBy(_._2, false).take(3)
     println("==========top3========")
     sortArr.foreach(println)
     //transform要求返回RDD类型，故将rdd返回
     rdd
  })
   
   //注意:上面的代码没有涉及到DStream的Action操作,会报错:No output operations registered, so nothing to execute所以下面的代码不能省略，虽然没有意义
   result.print()

   //ssc需要如下形式停止
   ssc.start()
   ssc.awaitTermination()//等待终止

   sc.stop()
   spark.stop()
   }
}
```

