# SparkSQL

# 概述

## SparkSQL是什么

SparkSQL是Spark用于处理结构化数据的一个模块。

## SparkSQL数据处理方式

SparkSQL提供了两种数据分析方式，分别是代码式与SQL式。

代码式风格步骤清晰，处理粒度细致，对于代码能力要求较高。

SQL式风格表达简洁，入门门槛较低，但对于复杂任务交困难。

## 应用场景

处理结构化数据与较为规范的半结构化数据。

# SparkSQL的数据抽象

## DataFrame

Spark1.3之后开始支持的数据抽象，实际上是一个特殊的RDD，取消了泛型约束，可以进行SQL操作。

等同于一个分布式的表。

`DataFrame = RDD - 泛型 + Schema + SQL操作 + 优化`

## DataSet

Spark1.6之后开始支持的数据抽象，是一个特殊的DataFrame，比DataFrame增加泛型。

同样等同于一个分布式的表。

`DataSet = DataFrame + 泛型`

## RDD/DF/DS对比

![image-20230129110515332](http://image.coolcode.fun/images/202301291105430.png)

如图所示，**DataFrame比RDD多了字段，DataSet比DataFrame多了泛型。**

# DataFrame与DataSet的创建

## 通过RDD进行转换

### 样例类+反射创建DF

```scala
//personRDD中存的是样例类Person的对象
case class Person(id:Int,name:String,age:Int)
val personRDD: RDD[Person] = lineRDD.map(arr=>Person(arr(0).toInt,arr(1),arr(2).toInt))

//toDF()方法需要隐式转换
import sparkSession.implicits._
//RDD使用toDF()方法转换为DF
val personDF: DataFrame = personRDD.toDF()
```

### 手动指定Schema创建DF

```scala
//对应列较少的情况可以不将数据封装为样例类，使用Row对象进行封装
val rowRDD: RDD[Row] = lineRDD.map(arr=>Row(arr(0).toInt,arr(1),arr(2).toInt))

//根据参数要求准备Schema
val schema:StructType = StructType(Seq(
   StructField("id", IntegerType, true),
   StructField("name", StringType, true),
   StructField("age", IntegerType, true)))
//使用SparkSession的createDataFrame方法，将要转换的RDD与Schema传入，转换为DF
val personDF: DataFrame = sparkSession.createDataFrame(rowRDD,schema)
```

### 手动指定列名创建DF

```scala
//对应列较少的情况可以不将数据封装,直接在转换时指定列名
val tupleRDD: RDD[(Int, String, Int)] = lineRDD.map(arr=>((arr(0).toInt,arr(1),arr(2).toInt)))

//toDF()方法需要隐式转换
import sparkSession.implicits._
//将RDD转为DF,并手动指定列名
val personDF: DataFrame = tupleRDD.toDF("id","name","age")
```

### 通过读取文本创建

DataFrame/DataSet都可以直接通过读取普通文本创建,但是没有完整的约束。

### 通过读取json/parquet文件创建

DataFrame/DataSet也可以直接通过读取json/parquet文件创建,有约束。

## RDD/DataFream/DataSet之间相互转换

![image-20230129110815372](http://image.coolcode.fun/images/202301291108488.png)

### **RDD => DataFrame**

```
val personDF: DataFrame = personRDD.toDF()              
```

### **RDD => DataSet**

```
val personDS: Dataset[Person] = personRDD.toDS()              
```

### **DataFrame => RDD**

```
val personRDD: RDD[Row] = personDF.rdd              
```

### **DataFrame => DataSet**

```
val personDS: Dataset[Person] = personDF.as[Person]              
```

### **DataSet => RDD**

```
val personRDD: RDD[Person] = personDS.rdd              
```

### **DataSet => DataFrame**

```
val personDF: DataFrame = personDS.toDF()              
```

*Person是样例类

# **SparkSQL操作多种数据源**

## **从不同数据源读取数据**

```scala
//Json
spark.read.json("D:\\data\\output\\json").show()
//csv 需要指定列名
spark.read.csv("D:\\data\\output\\csv").toDF("id","name","age").show()
//parquet
spark.read.parquet("D:\\data\\output\\parquet").show()
//数据库
val properties = new Properties()
properties.setProperty("user","root")
properties.setProperty("password","root")
spark.read.jdbc("jdbc:mysql://localhost:3306/database?characterEncoding=UTF-8","t_person",properties).show
```

## **向不同数据源写入数据**

```scala
//写入数据前需要先合并DataFrame的分区
val oneDF: Dataset[Row] = personDF.repartition(1)
//Json
oneDF.write.json("D:\\data\\output\\json")
//csv
oneDF.write.csv("D:\\data\\output\\csv")
//parquet
oneDF.write.parquet("D:\\data\\output\\parquet")

//数据库
val properties = new Properties()
properties.setProperty("user","root")
properties.setProperty("password","root")   
oneDF.write.mode(SaveMode.Overwrite).jdbc("jdbc:mysql://localhost:3306/database?characterEncoding=UTF-8","t_person",properties)
```

# **SparkSQL自定义函数**

```scala
import org.apache.spark.SparkContext
import org.apache.spark.sql._

/**
  * Desc 使用SparkSQLUDF函数,将每一行数据转换成大写
  */

object UDFTest {
  def main(args: Array[String]): Unit = {
    //1.创建SparkSession
    val spark: SparkSession = SparkSession.builder().master("local[*]").appName("sparkSQL").getOrCreate()
    val sc: SparkContext = spark.sparkContext
    sc.setLogLevel("WARN")
      
    //2.读取文件
    val df: DataFrame = spark.read.text("D:\\data\\udf.txt")
    df.show()
    /*
    +----------+
    |     value|
    +----------+
    |helloworld|
    |       abc|
    |     study|
    | smallWORD|
    +----------+
     */
      
    //3.注册表
    df.createOrReplaceTempView("t_word")
      
    //4.自定义udf函数，完成大写转换
    //register方法的第一个参数为自定义函数名称,第二个参数为函数的处理逻辑
    spark.udf.register("smallToBig",(word:String)=>word.toUpperCase)
    
    //5.使用自定义的函数
    spark.sql("select value,smallToBig(value) from t_word").show()
    /*
    +----------+---------------------+
    |     value|UDF:smallToBig(value)|
    +----------+---------------------+
    |helloworld|           HELLOWORLD|
    |       abc|                  ABC|
    |     study|                STUDY|
    | smallWORD|            SMALLWORD|
    +----------+---------------------+
     */

    sc.stop()
    spark.stop()
  }
}

```

# **案例：单词个数统计**

```scala
import org.apache.spark.SparkContext
import org.apache.spark.sql.{DataFrame, Dataset, SparkSession}

/**
  * Desc 使用SparkSql完成WordCount
  */

object WordCount {
  def main(args: Array[String]): Unit = {
    //1.创建SparkSession
    val spark: SparkSession = SparkSession.builder().master("local[*]").appName("sparkSQL").getOrCreate()
    val sc: SparkContext = spark.sparkContext
    sc.setLogLevel("WARN")
    import spark.implicits._
    
      //2.读取文件
    val dataDF: DataFrame = spark.read.text("D:\\data\\words.txt")
    val dataDS: Dataset[String] = spark.read.textFile("D:\\data\\words.txt")
    
    //3.每一行按照空格进行切分并压平
    //dataDF.flatMap(_.split(" ")) 
    //_表示每一行,DF没有泛型,不能确定数据为是String类型,所以不能调用split方法
    val wordDS: Dataset[String] = dataDS.flatMap(_.split(" "))
    //_表示每一行,但是DS有泛型,数据为String,所以能调用split方法
    /*
    wordDS：
    +-----+
    |value|
    +-----+
    |hello|
    |   me|
    |  you|
    |  her|
      ....
     */
      
      
    //4.使用SQL风格完成WordCount
    //4.1注册表
    wordDS.createOrReplaceTempView("t_word")
    //4.2编写sql
    val sql:String =
      """
        |select value ,count(*) counts
        |from t_word
        |group by value
        |order by counts desc
      """.stripMargin
    spark.sql(sql).show
    /*
    +-----+------+
    |value|counts|
    +-----+------+
    |hello|     4|
    |  her|     3|
    |  you|     2|
    |   me|     1|
    +-----+------+
     */

      
    //5.使用DSL风格完成WordCount
    wordDS.groupBy("value").count().orderBy($"count".desc).show()
    /*
    +-----+-----+
    |value|count|
    +-----+-----+
    |hello|    4|
    |  her|    3|
    |  you|    2|
    |   me|    1|
    +-----+-----+
     */

    sc.stop()
    spark.stop()
  }
}

```

​        