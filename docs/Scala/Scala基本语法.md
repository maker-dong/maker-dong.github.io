# Scala基础语法

#  **Scala简介**

- 运行在JVM之上

- - Scala编译为.class文件，运行在JVM上

- 多范式

- - 面向对象
  - 面向函数

Scala代码可以转换成Java代码，Java代码不可以转换成Scala代码

> 函数式编程思想：函数也可以当成一个对象，函数可以作为参数进行传递

## **Scala优点**

- 表达能力强，开发速度快
- 兼容Java，可以使用Java的类库

# **变量**

## **变量定义**

```
 val 变量名:变量类型 = 初始值 var 变量名:变量类型 = 初始值
```

- val定义的是不可重新赋值的变量

- var定义的是可重新赋值的变量

### **类型推断**

Scala可以自动根据变量的值来自动推断变量的类型，但必须有初始值才能省略编写类型。

```
val 变量名 = 初始值
```

### **惰性赋值**

当有一些变量保存的数据较大时，但是不需要马上加载到JVM内存。可以使用惰性赋值来提高效率。

使用惰性赋值，变量在定义时不加载到内存中，**在调用时才加载到内存中**。

```
lazy val 变量名 = 表达式 
```

# 数据类型

Scala中的数据类型与Java基本一致，但数据类型首字母要大写。

| 基础类型 | 类型说明                 |
| -------- | ------------------------ |
| Byte     | 8位带符号整数            |
| Short    | 16位带符号整数           |
| **Int**  | 32位带符号整数           |
| Long     | 64位带符号整数           |
| Char     | 16位无符号Unicode字符    |
| String   | Char类型的序列（字符串） |
| Float    | 32位单精度浮点数         |
| Double   | 64位双精度浮点数         |
| Boolean  | true或false              |

## **类型的层次结构**

![image-20230607155517963](http://image.coolcode.fun/images/202306071555037.png)

- Any		：所有类型的父类
- Null		：所有引用类型的子类
- Unit		：相当于Java中的void，当返回值为空的时候,返回的就是Unit，但Unit的实例就是一个括号()
- Nothing	：是所有类型的子类

## **Scala字符串**

### **使用双引号**

```
val/var 变量名 = “字符串”
```

### **使用插值表达式**

```
val/var 变量名 = s"${变量/表达式}字符串"
```

示例：

```scala
val name = "zhangsan" val info = s"name=${name}" info:name = "zhangsan"
```

### **使用三引号**

有大段的文本需要保存，就可以使用三引号来定义字符串。

```
val/var 变量名 = """字符串"""
```

示例：

```scala
val sql = """select
	| *     
	| from     
	|     t_user     
	| where     
	|     name = "zhangsan""""  
```

三引号字符串从第一个三引号开始到第二个三引号结束，不要出现重复, 比如 """ aaa """ bbb """

# 运算符

Scala 中的运算符, 基本上和Java一样，但是

- **Scala中没有++与--**
- **Scala中直接使用 == 与 != 对值进行比较，与equals方法一致；比较引用值使用eq（equals 比较值，eq 比较引用地址）**

**== 在对象是null的时候 调用eq, 对象非null 的时候调用equals**

| 类别       | 操作符                        |
| ---------- | ----------------------------- |
| 算术运算符 | +、-、*、/、%(加减乘除和取模) |
| 关系运算符 | >、<、==、!=、>=、<=          |
| 逻辑运算符 | &&、\|\|、!                   |
| 位运算符   | &、\|\|、^、<<、>>            |

# **Scala条件表达式**

## **块表达式**

- 使用{}括起来的一部分代码
- 块表达式有返回值，返回值就是最后一个表达式的值

### **if表达式**

if表达式是有返回值的，返回值的内容就是块表达式的内容

```scala
 val result = if(sex == "male") 1 else 0 result: Int = 1              
```

Scala中没有三元表达式，如果要使用三元表达式则需要使用if的形式

# **Scala循环**

## **for表达式**

```scala
for(变量 <- 表达式/数组/集合) {
	循环体
}
```



```scala
for(i <- 1 to 10){
	println(i)
}
```

### **简单循环**

```scala
val nums = 1.to(10)    // 万物皆对象, 1 也可以看成是一个对象 to就是这个对象的方法
nums: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
for(i <- nums) println(i) 
```

### **嵌套循环**

```scala
for(i <- 1 to 3; j <- 1 to 5) {print(j)}
```

用分号分割每一层循环，从左到右是从外到内的顺序

### **守卫**

在for循环的条件后直接添加if表达式，满足if条件的变量才能进入循环体内

```scala
for(i <- 表达式/数组/集合 if 表达式) {
	循环体
}
```

示例：

```scala
// 添加守卫，打印能够整除3的数字
for(i <- 1 to 10 if i % 3 == 0) println(i)
```

### **for推导式**

使用yield关键字修饰的for循环会把循环体的每一次返回组装起来统一返回

```scala
// for推导式：for表达式中以yield开始，该for表达式会构建出一个集合
val v = for(i <- 1 to 10) yield i * 10
```

## **while循环**

```scala
while(判断表达式){
	循环体
}
```

### **break&continue**

**Scala中没有break与continue关键字**

使用scala.util.control包的Break类的breable和break方法实现

#### **实现break**

```scala
// 导入scala.util.control包下的Break 
import scala.util.control.Breaks._ 
breakable{
	for(i <- 1 to 100) {
		if(i >= 50) break()
		else println(i)
	}
}    
```

breakable包住for循环, 那么跳出的时候, 就把for循环也跳出了, 就向下继续执行了, 就实现了类似java break的功能.

#### 实现continue

# **Scala方法**

## **方法定义**

```scala
def methodName (参数名:参数类型, 参数名:参数类型) : [return type] = {
	// 方法体：一系列的代码
}           
```

- 参数列表的参数类型不能省略
- 方法定义可以省略返回值类型
- 方法体的表达式就是方法的返回值

**注意：递归方法不可以省略**

## **方法参数**

### **默认参数**

定义方法时，可以给参数默认值，如果不传参可以使用默认值进行计算。

```scala
// x，y带有默认值为0  
def add(x:Int = 0, y:Int = 0) = x + y 
add()
```

### **带名参数**

调用方法时，可以指定参数的名称进行调用

```scala
def add(x:Int = 0, y:Int = 0) = x + y 
add(x=1)
```

### **变长参数**

如果方法参数不固定，可以定义一个方法的参数是变长参数

```scala
def 方法名(参数名:参数类型*):返回值类型 = {
	方法体
}
```

注意：

- 变长参数接收的是一个个Int，不能直接传一个Array，可以使用：`add(ar:_*)`
- 变长参数必须是最后一个参数

## **方法调用**

### **后缀调用法**

```
对象名.方法名(参数)              
```

### **中缀调用法**

```
对象名 方法名 参数              
```

示例:

```
1 to 10 
相当于 
1.to(10)              
```

### **花括号调用法**

方法只有一个参数可以使用花括号调用法

```scala
Math.abs{
	// 表达式1
    // 表达式2
}              
```

### **无括号调用法**

方法没有参数可以使用无括号调用发

```scala
def m3()=println("hello") 
m3              
```

# **Scala函数**

## **定义函数**

```scala
val 函数变量名 = (参数名:参数类型, 参数名:参数类型....) => 函数体              
```

**函数是一个对象所以函数可以调用方法**

方法无法赋值给对象, 但是函数定义的时候, 就是直接赋值给了对象的

方法在方法区内，函数是一个对象，运行的时候在堆内存中。

### **方法转换为函数**

```scala
def add(x:Int,y:Int)=x+y
add: (x: Int, y: Int)Int

val a = add _
a: (Int, Int) => Int = <function2>              
```

# **数组**

与Java中的数组一样，存放一种相同数据类型的数据。

## **定长数组**

长度不允许改变，元素内容允许改变

### **定义数组**

```scala
// 通过指定长度定义数组 
val/var 变量名 = new Array[元素类型](数组长度)

// 用元素直接初始化数组
val/var 变量名 = Array(元素1, 元素2, 元素3...)  
```

### **泛型**

在scala中，数组的泛型使用`[ ]`来指定(java `<>`)

### **获取**

使用`( )`来获取元素(java `[]`)

## **变长数组**

长度可变，元素可替换

创建变长数组，需要提前导入`ArrayBuffer`类

```scala
import scala.collection.mutable.ArrayBuffer
```

### **定义数组**

```scala
//创建空的ArrayBuffer变长数组，语法结构：
val/var a = new ArrayBuffer[元素类型]()

//创建带有初始元素的ArrayBuffer
val/var a = ArrayBuffer(元素1，元素2，元素3....)
```

### **添加**

- 使用+=追加元素

- 使用++=追加一个数组

### **删除**

- 使用-=删除元素（后跟元素本身）

- 使用--=删除一个数组

- 如果有两个相同的元素，则会删除前面的

# **元组**

一个括号括起来的元素列表，其中元素类型可以不同

**元组创建后就确定了，不能追加元素也不可以修改或删除元素。**

**元组不能使用for循环遍历**

## **定义元组**

```scala
 // 使用括号来定义元组 
 val/var 元组 = (元素1, 元素2, 元素3....) 
 
 //val/var 元组 = 元素1->元素2
 val/var 元组 = 元素1->元素2
```

## **访问元组**

使用`_1`、`_2`来访问元组中的元素。

`_1`、`_2`...是成员

注意：**元组的获取从1开始，不是0！！！**

```scala
val a = "zhangsan" -> "male"
a: (String, String) = (zhangsan,male) 

// 获取第一个元素 
scala> a._1 
res41: String = zhangsan 

// 获取第二个元素
scala> a._2 
res42: String = male 
```

# **列表**

- 保存同类型的元素
- 内容可重复
- 记录内容有序

## **不可变列表**

- 长度不可变
- 元素不可改变

```scala
val/var 变量名 = List(元素1, 元素2, 元素3...)
```

## **可变列表**

- 长度可变
- 元素可以改变

要使用可变列表，先要导入

```scala
import scala.collection.mutable.ListBuffer              

//ListBuffer可以用new创建 
val/var 变量名 = new ListBuffer[Int]() 
val/var 变量名 = ListBuffer(元素1，元素2，元素3...)    
```

### **添加**

- 使用+=追加元素

- 使用++=追加一个数组


### **删除**

使用-=删除元素（后跟元素本身）

使用--=删除一个数组

如果有两个相同的元素，则会删除前面的

## **列表常用操作**

- isEmpty	：判断List是否为空
- ++		：拼接两个List
- head		：取List的首个元素
- tail		：却List除头元素外的剩余元素
- reverse	：List反转
- take(x)	：取前x个
- drop(x)	：去除前x个取剩下的
- flaten	：扁平化，去掉一层嵌套（想要去掉多层嵌套需要使用链式调用）注意：使用flaten要求List一定要平衡
- zip()		：拉链，l1.zip(l2)，将l1与l2中的元素一一对应组合成元组
- unzip		：拉开，将zip后的元组解开成两个List，用元组封装
- toString	：将List转换成String对象，包括（）与，
- mkString	：将List中的元素转换为字符串，mkString(，)参数表示分隔符，无参则没有分隔符
- union	：求并集，l1.union(l2) 不去重
- intersect	：求交集，l1 intersect l2
- diff		：求差集，l1 diff l2 求出l1中存在但在l2中不存在的元素
- distinct	：对List去重

# **集**

- 元素不可重复
- 存储元素无序
- 没有索引
- 存储相同类型的元素

## **不可变集**

- 长度不可变
- 元素不可变

```scala
val/var 变量名 = Set[类型]()
val/var 变量名 = Set(元素1, 元素2, 元素3...)
```

###  **操作**

- size		：获取集的大小
- 遍历集
- `+`：添加一个元素，结果生成一个新的Set
- `++`：拼接两个集，生成一个新的Set

## **可变集**

```scala
import scala.collection.mutable.Set              

val/var 变量名 = Set[类型]()
val/var 变量名 = Set(元素1, 元素2, 元素3...)              
```

### **操作**

- `+=`		：给自身添加一个元素
- `++=`		：给自身添加一个集
- `-=`		：删除一个元素
- `--=`		：删除一个集

# **映射**

Map，由键值对组成的集合，每一个元素都是一个元组。

## **不可变Map**

```scala
val/var map = Map(键->值, 键->值, 键->值...)	// 推荐，可读性更好
val/var map = Map((键, 值), (键, 值), (键, 值), (键, 值)...) 
```

### **获取元素**

```scala
map.get(x) map(x) //x是键，不是索引
```

## **可变Map**

```scala
import scala.collection.mutable.Map              

val/var map = Map(键->值, 键->值, 键->值...)	// 推荐，可读性更好
val/var map = Map((键, 值), (键, 值), (键, 值), (键, 值)...)
```

### **添加元素**

```scala
map += (key -> value) 
map ++= Map(key -> value.key -> value...)              
```

### **删除元素**

```scala
map -= (key) 
map --= Map(key,key...)              
```

# **迭代器iterator**

```scala
val ite = a.iterator
ite: Iterator[Int] = non-empty iterator 
while(ite.hasNext) {
	println(ite.next)
}              
```

注意：**迭代器只能用一次, 内部指针只走一次, 走到最后就结束了, 如果再次使用需要再取得一个新的迭代器**

