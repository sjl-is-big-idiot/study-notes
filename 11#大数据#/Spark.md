# 1. spark概述



<font color="red">***注意：Spark3.0、Spark3.1是由Scala2.12预编译，但是Spark 3.2+是采用Scala 2.13预编译。***</font>

***注意，在Spark 2.0之前，Spark的主要编程接口是弹性分布式数据集（RDD）。在Spark 2.0之后，RDD被Dataset所取代，Dataset与RDD相似，但在后台进行了更丰富的优化。RDD接口仍然受支持，但DataSet比RDD性能更好。***

https://spark.apache.org/docs/3.0.2/rdd-programming-guide.html

https://spark.apache.org/docs/3.0.2/sql-programming-guide.html



注意，默认情况下Spark的security是关闭的。这意味着容易受到攻击。参考：https://spark.apache.org/docs/3.0.2/security.html

## 1.1 交互式命令行

Spark提供了交互式命令行以便用户可以学习Spark的API。目前（2023.01.29）Spark提供了2种交互式命令行：

- Scala，运行在JVM中
- Python，

### Scala

Spark的主要抽象为DataSet（原为RDD）。可以从Hadoop InputFormats（如HDFS文件）或通过转换其他DataSet来创建一个新的DataSet。

#### 基础操作

```scala
./bin/spark-shell
// 1.创建一个DataSet对象
scala> val textFile = spark.read.textFile("D:\\services\\hadoop-3.2.2\\README.txt")
textFile: org.apache.spark.sql.Dataset[String] = [value: string]

// 2.通过action算子获取DataSet中的数据 
scala> textFile.count() // Number of items in this Dataset
res0: Long = 126 // May be different from yours as README.md will change over time, similar to other outputs

scala> textFile.first() // First item in this Dataset
res1: String = # Apache Spark

// 3.通过transform算子根据旧的DataSet创建出一个新的DataSet
scala> val linesWithSpark = textFile.filter(line => line.contains("Spark"))
linesWithSpark: org.apache.spark.sql.Dataset[String] = [value: string]

// 4.通过action算子获取新的DataSet中的数据
scala> textFile.filter(line => line.contains("Spark")).count() // How many lines contain "Spark"?
res3: Long = 15
```

#### 高级操作

```scala
// 这首先将一行映射到一个整数值，创建一个新的数据集。
// 对该数据集调用reduce以查找最大的单词计数。map和reduce的参数是Scala函数文本（闭包），可以使用任何语言特性或Scala/Java库。例如，我们可以很容易地调用其他地方声明的函数。我们将使用Math.max（）函数使代码更容易理解：
scala> textFile.map(line => line.split(" ").size).reduce((a, b) => if (a > b) a else b)
res4: Long = 15

scala> import java.lang.Math
import java.lang.Math

scala> textFile.map(line => line.split(" ").size).reduce((a, b) => Math.max(a, b))
res5: Int = 15

scala> val wordCounts = textFile.flatMap(line => line.split(" ")).groupByKey(identity).count()
wordCounts: org.apache.spark.sql.Dataset[(String, Long)] = [value: string, count(1): bigint]

scala> wordCounts.collect()
res6: Array[(String, Int)] = Array((means,1), (under,2), (this,3), (Because,1), (Python,2), (agree,1), (cluster.,1), ...)


```

#### 缓存

Spark支持将DataSet跨集群缓存。当访问“热数据“时更加高效。

```scala
scala> linesWithSpark.cache()
res7: linesWithSpark.type = [value: string]

scala> linesWithSpark.count()
res8: Long = 15

scala> linesWithSpark.count()
res9: Long = 15
```



### Python

TODO

```python
./bin/pyspark
>>> textFile = spark.read.text("README.md")
>>> textFile.count()  # Number of rows in this DataFrame
126

>>> textFile.first()  # First row in this DataFrame
Row(value=u'# Apache Spark')
>>> linesWithSpark = textFile.filter(textFile.value.contains("Spark"))
>>> textFile.filter(textFile.value.contains("Spark")).count()  # How many lines contain "Spark"?
15

>>> from pyspark.sql.functions import *
>>> textFile.select(size(split(textFile.value, "\s+")).name("numWords")).agg(max(col("numWords"))).collect()
[Row(max(numWords)=15)]
>>> wordCounts = textFile.select(explode(split(textFile.value, "\s+")).alias("word")).groupBy("word").count()
>>> wordCounts.collect()
[Row(word=u'online', count=1), Row(word=u'graphs', count=1), ...]

>>> linesWithSpark.cache()

>>> linesWithSpark.count()
15

>>> linesWithSpark.count()
15
```



## 1.2 应用程序调用Spark API

Scala + Maven，Scala + SBT 自己去研究

```scala
package org.spark.sjl

/**
 * SimpleApp.scala
 * */
import org.apache.spark.sql.SparkSession
object SimpleApp {
  def main(args: Array[String]): Unit = {
    val logFile = "D:\\services\\spark-3.0.2-bin-hadoop3.2\\README.md"
    val spark = SparkSession.builder().master("local[2]").appName("Simple App").getOrCreate()
    val logData = spark.read.textFile(logFile)
    val numAs = logData.filter(line => line.contains("a")).count()
    val numBs = logData.filter(line => line.contains("b")).count()

    println(s"Lines with a: $numAs, Lines with b: $numBs")
    spark.stop()
  }
}
```

我们调用SparkSession.builder来构造一个SparkSession，然后设置应用程序名称，最后调用getOrCreate来获取SparkSession实例。

pom.xml中的部分依赖如下：	

![image-20230131151642518](Spark.assets/image-20230131151642518.png)

![image-20230131151329279](Spark.assets/image-20230131151329279.png)

打成jar包

使用IDEA或Maven打包，也可以直接在项目根目录下使用`mvn package`。

使用`spark-submit`提交作业

![image-20230131154338987](Spark.assets/image-20230131154338987.png)

报错提示找不到主类SimpleApp。解决方案：**主类写为全路径**"org.spark.sjl.SimpleApp"。



## 1.3 RDD编程

### 概述

https://www.jianshu.com/p/6411fff954cf

Spark提供了两种数据抽象：

- RDD，Resilient Distributed Dataset，弹性分布式数据集。每一个RDD包含的数据被存储在系统的不同节点上。逻辑上我们可以将RDD理解为数组，数组中的每个元素代表一个分区（Partition）。

  RDD可以自动从故障节点中恢复。TODO，怎么做到的

- 共享变量，Share Variables。默认情况下，如果在一个[算子](https://so.csdn.net/so/search?q=算子&spm=1001.2101.3001.7020)函数中使用到了某个外部的变量，那么这个变量的值会被拷贝到每个task中。此时每个task只能操作自己的那份变量副本。如果多个task想要共享某个变量，那么这种方式是做不到的。

  - 广播变量，broadcast variables。Broadcast Variable会将使用到的变量，仅仅为每个节点拷贝一份，而不会为每个task都拷贝一份副本，因此其最大的作用，就是减少变量到各个节点的网络传输消耗，以及在各个节点上的内存消耗
    通过调用SparkContext的broadcast()方法，针对某个变量创建广播变量。

    注意:广播变量，是只读的，然后在算子函数内，使用到广播变量时，每个节点只会拷贝一份副本。

  - 累加变量，accumulator。Spark提供的Accumulator，主要用于多个节点对一个变量进行共享性的操作。正常情况下在Spark的任务中，由于一个算子可能会产生多个task并行执行，所以在这个算子内部执行的 聚合计算都是局部的，想要实现多个task进行全局聚合计算，此时需要使用到Accumulator这个共享的累加变量。

    注意:Accumulator只提供了累加的功能。在task只能对Accumulator进行累加操作，不能读取它的值。 只有在Driver进程中才可以读取Accumulator的值。

### 连接到Spark

通常我们可以写Scala、Java、Python程序来连接到Spark。

#### Scala

Spark 3.0.2 默认采用Scala 2.12构建（当然可以手动通过其他版本的Scala来构建Spark 3.0.2）。因此写Scala的application，需要使用Scala 2.12.X（为了兼容性）。

连接到Spark需要如下Maven依赖：

```properties
groupId = org.apache.spark
artifactId = spark-core_2.12
version = 3.0.2
```

如果还有访问HDFS的需求，则需要增加`hadoop-client`依赖：

```properties
groupId = org.apache.hadoop
artifactId = hadoop-client
version = <your-hdfs-version>
```

最后，在写代码是通过，如下语句导入Spark API的类

```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkConf
```

<font color="red">***注意，在Spark 1.3.0之前，您需要显式导入`org.apache.Spark.SparkContext._`以启用基本的隐式转换。***</font>

#### Java

TODO

#### Python

TODO

### 初始化Spark



#### Scala

写Spark application的第一步就是创建SparkConf对象，此对象包含此application的信息。

写Spark application的第二步就是创建SparkContext对象，此对象用于告诉Spark应该要如何去访问集群（local、standalone、Mesos、Hadoop、Kubernetes）

<font color="red">***注意，每个JVM同时只能有一个SparkContext活跃，如果要创建一个新的SparkContext，必须要stop()之前是SparkContext***</font>

```scala
val conf = new SparkConf().setAppName(appName).setMaster(master)
val sc = new SparkContext(conf)
......
sc.stop()
```



#### Java

TODO

#### Python

TODO

### 使用Spark的shell

#### Scala

在`spark-shell`中已经预先创建了一个`SparkContext`，名为`sc`，不用在手动创建SparkContext对象了（而且创建了也不生效）。

使用`--master`指定spark连接到哪个master。

```bash
$ ./bin/spark-shell --master local[4]
```

使用`--jars` 将 code.jar添加到`classpath`

```bash
$ ./bin/spark-shell --master local[4] --jars code.jar
```

使用`--packages` 指定所需的依赖

```bash
$ ./bin/spark-shell --master local[4] --packages "org.example:example:0.1"
```

更多的使用方式`spark-shell --help`

#### Python

TODO

### RDD，弹性分布式数据集

spark提出了一个概念RDD，RDD是不可变的、可并行操作的、容错的、分区的数据集合。

有两个方式可以创建RDD：

- 并行化已存在driver中的数据集合
- 引用外部数据中的数据

#### 并行化数据集，Parallelized Collections

##### Scala

调用`SparkContext`对象的`parallelize()`方法

```scala
val data = Array(1,2,3,4,5)
val distData = sc.parallelize(data)
```

parallelize()方法有一个重要的参数`partitions`，此参数决定将RDD切分为几份。Spark会为集群中的每个partition运行一个task来处理。通常，集群中每个CPU需要2-4个分区。通常，Spark会根据集群自动设置分区数。但是，您也可以通过将其作为要并行化的第二个参数（例如`sc.paralleize(data，10)`）来手动设置它。

##### Java

TODO

##### Python

TODO

### 外部数据集（External DataSets）

Spark可以从任何Hadoop支持的存储系统（如，本地文件系统、HDFS、Cassandra、HBase、Amazon S3等）中的数据来创建RDD。

Spark支持的文件格式为textFile、SequenceFile和其他所有Hadoop支持的InputFormat。



```scala
scala> val distFile = sc.textFile("data.txt")
distFile: org.apache.spark.rdd.RDD[String] = data.txt MapPartitionsRDD[10] at textFile at <console>:26
```

`textFile()`将文本文件中的每一行读取为集合中的一个元素。

注意：

- 使用本地文件系统路径时，需要保证此文件在所有worker节点上都是存在的，且可访问。可以将文件拷贝到所有worker节点，或者使用network-mounted shared file system。
- Spark中的textFile()等类似的方法还支持目录、压缩文件、通配符。如`textFile("/my/dirctory")`, `textFile("/my/directory/*.txt")`, `textFile("/my/directory/*.gz")`
- textFile()的第二个参数可以控制此文件切分为多少个partition。默认情况下，Spark为文件的每个块（HDFS中默认块大小为128MB）创建一个partition。通过textFile()中第二参数可以手动指定此RDD的partition个数，硬性要求partition个数>=块数量。

除了文本文件，Spark的Scala API还支持如下几种数据格式：

- `SparkContext.wholeTextFiles` 读取指定目录下的所有文本文件，以（filename, content)对的形式访问。第二个参数控制最小partition数量。
- `SparkContext.sequenceFile[k,v]` 其中K和V是文件中键和值的类型。k,v必须是Hadoop的 Writable接口的子类。
- `SparkContext.hadoopRDD` 其他Hadoop InputFormat格式使用此方法
- `RDD.saveAsObjectFile` 和`SparkContext.objectFile`可以将RDD存储为序列化的Java对象。效率不如Avro，但是胜在使用简单。

### RDD操作

RDD支持两种操作：

- transformation，转换。从现有RDD创建新RDD。如map()
- action，行动。对RDD执行计算之后将结果返回给driver。如reduce()。

Spark中所有的transformation都是惰性的，它们不会立即计算结果，只需要记住在此RDD上应用了哪些transformation操作即可。只有当遇到action算子时，才会执行这些transformation算子，并且返回action算子的执行结果给driver。此设计使得Spark更加高效。

> RDD提供了一个抽象的数据模型，不比担心底层数据的分布式特性，只需将具体的应用逻辑表达为一系列转换操作（函数），不同RDD之间的转换操作之间还可以形成依赖关系，进而实现管道化，从而避免了中间结果的存储，大大降低了数据复制、磁盘IO和序列化开销，

每次执行action算子时都会重新计算RDD，如果需要多次使用则十分影响效率，通过RDD 持久化可以提升Spark的计算效率。

#### 基础操作

##### Scala

```scala
val line = sc.textFile("data.txt")
val lineLengths = line.map(s => s.length)
val totalLength = lineLengths.reduce((a, b) => a+b)
```

第一步，创建了一个名为line的RDD，实际只是line指向了"data.txt"。实际上还并没有读取"data.txt"内容存到line变量中。

第二步，使用了map算子

第三步，使用了reduce算子，此时Spark才将计算分解为多个task，分布到集群中的节点上去运行，每个节点执行自己的map和本地reduce，然后将结果返回给driver，由driver汇总。

如果后面想要复用lineLengths，则

```scala
lineLengths.persist()
```

否则，每次遇到action算子时，都会计算一遍lineLengths。

#### 将函数传递给Spark

Spark的API严重依赖于在集群上运行的驱动程序中传递函数。两种方式：

- Anonymous function synctax，匿名函数。

- Static method，静态方法。

  ```scala
  object MyFunctions {
    def func1(s: String): String = { ... }
  }
  
  myRdd.map(MyFunctions.func1)
  ```

  

- x

- 



```scala
class MyClass {
  def func1(s: String): String = { ... }
  def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(func1) }
}
```

doStuff方法的作用等价于``rdd.map(x => this.func1(x))`.`

```scala
class MyClass {
  val field = "Hello"
  def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(x => field + x) }
}
```

等价于`rdd.map(x => this.filed+x)`，更好的方式是使用一个本地变量接收`this.filed`的值。

```scala
def doStuff(rdd: RDD[String]): RDD[String] = {
  val field_ = this.field
  rdd.map(x => field_ + x)
}
```



#### 闭包

Spark的一个难点在于，跨集群执行代码时，变量和方法的范围和生命周期。

##### 示例

```scala
var counter = 0
var rdd = sc.parallelize(data)

// Wrong: Don't do this!!
rdd.foreach(x => counter += x)

println("Counter value: " + counter)
```

如果是在同一个JVM中执行此代码没什么问题，但是如果在集群中的多个不同JVM中执行代码就会出现问题了

##### local模式和cluster模式

要执行job，Spark会将要处理的RDD操作分解到task中，每个task属于一个executor。在执行task之前，Spark会计算task的闭包。这里的闭包是指必须对executor可见的变量或方法（因为executor要对此RDD执行计算），如上个示例中的`foreach()`。然后这个闭包就会被序列化并发送给每个executor。



发送给每个executor的闭包中的变量和方法都是driver中的副本。如`foreach()`中的counter这与driver中的counter是两个不同的变量了。每个task计算时，都是指在自己的那个counter上进行`+=`，driver中的counter并没有发生改变。最终counter的值仍为0。

在本地模式下，在某些情况下，foreach函数实际上将在与驱动程序相同的JVM中执行，并将引用相同的counter，并更新它。要集群模式中实现此功能，应该使用共享变量中的累加器。

##### 打印RDD中的元素

```scala
// 每个task将结果返回给driver之后，遍历打印
// 可能存在的问题是当RDD过大时，会出现OOM
rdd.collect().foreach(println)
// 只打印部分数据
rdd.take(100).foreach(println)
```



#### 使用键值对

大多数Spark操作可以在`RDD[T]`上工作，但少数特殊操作仅适用于键值对的RDD。最常见的是分布式“shuffle”操作，例如通过键对元素进行分组或聚合。

例如，以下代码对键值对使用`reduceByKey`操作来计算文件中每行文本出现的次数：

```scala
val lines = sc.textFile("data.txt")
val pairs = lines.map(s => (s, 1))
val counts = pairs.reduceByKey((a, b) => a + b)
```

例如，我们还可以使用`counts.sortByKey()`按字母顺序排序，最后使用`counts.collect()`将它们作为对象数组返回驱动程序。

#### Transformation算子

下表列出常见的transformation算子。

| Transformation                                               | Meaning                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| **map**(*func*)                                              | Return a new distributed dataset formed by passing each element of the source through a function *func*. |
| **filter**(*func*)                                           | Return a new dataset formed by selecting those elements of the source on which *func* returns true. |
| **flatMap**(*func*)                                          | Similar to map, but each input item can be mapped to 0 or more output items (so *func* should return a Seq rather than a single item). |
| **mapPartitions**(*func*)                                    | Similar to map, but runs separately on each partition (block) of the RDD, so *func* must be of type Iterator<T> => Iterator<U> when running on an RDD of type T. |
| **mapPartitionsWithIndex**(*func*)                           | Similar to mapPartitions, but also provides *func* with an integer value representing the index of the partition, so *func* must be of type (Int, Iterator<T>) => Iterator<U> when running on an RDD of type T. |
| **sample**(*withReplacement*, *fraction*, *seed*)            | Sample a fraction *fraction* of the data, with or without replacement, using a given random number generator seed. |
| **union**(*otherDataset*)                                    | Return a new dataset that contains the union of the elements in the source dataset and the argument. |
| **intersection**(*otherDataset*)                             | Return a new RDD that contains the intersection of elements in the source dataset and the argument. |
| **distinct**([*numPartitions*]))                             | Return a new dataset that contains the distinct elements of the source dataset. |
| **groupByKey**([*numPartitions*])                            | When called on a dataset of (K, V) pairs, returns a dataset of (K, Iterable<V>) pairs. **Note:** If you are grouping in order to perform an aggregation (such as a sum or average) over each key, using `reduceByKey` or `aggregateByKey` will yield much better performance. **Note:** By default, the level of parallelism in the output depends on the number of partitions of the parent RDD. You can pass an optional `numPartitions` argument to set a different number of tasks. |
| **reduceByKey**(*func*, [*numPartitions*])                   | When called on a dataset of (K, V) pairs, returns a dataset of (K, V) pairs where the values for each key are aggregated using the given reduce function *func*, which must be of type (V,V) => V. Like in `groupByKey`, the number of reduce tasks is configurable through an optional second argument. |
| **aggregateByKey**(*zeroValue*)(*seqOp*, *combOp*, [*numPartitions*]) | When called on a dataset of (K, V) pairs, returns a dataset of (K, U) pairs where the values for each key are aggregated using the given combine functions and a neutral "zero" value. Allows an aggregated value type that is different than the input value type, while avoiding unnecessary allocations. Like in `groupByKey`, the number of reduce tasks is configurable through an optional second argument. |
| **sortByKey**([*ascending*], [*numPartitions*])              | When called on a dataset of (K, V) pairs where K implements Ordered, returns a dataset of (K, V) pairs sorted by keys in ascending or descending order, as specified in the boolean `ascending` argument. |
| **join**(*otherDataset*, [*numPartitions*])                  | When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (V, W)) pairs with all pairs of elements for each key. Outer joins are supported through `leftOuterJoin`, `rightOuterJoin`, and `fullOuterJoin`. |
| **cogroup**(*otherDataset*, [*numPartitions*])               | When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (Iterable<V>, Iterable<W>)) tuples. This operation is also called `groupWith`. |
| **cartesian**(*otherDataset*)                                | When called on datasets of types T and U, returns a dataset of (T, U) pairs (all pairs of elements). |
| **pipe**(*command*, *[envVars]*)                             | Pipe each partition of the RDD through a shell command, e.g. a Perl or bash script. RDD elements are written to the process's stdin and lines output to its stdout are returned as an RDD of strings. |
| **coalesce**(*numPartitions*)                                | Decrease the number of partitions in the RDD to numPartitions. Useful for running operations more efficiently after filtering down a large dataset. |
| **repartition**(*numPartitions*)                             | Reshuffle the data in the RDD randomly to create either more or fewer partitions and balance it across them. This always shuffles all data over the network. |
| **repartitionAndSortWithinPartitions**(*partitioner*)        | Repartition the RDD according to the given partitioner and, within each resulting partition, sort records by their keys. This is more efficient than calling `repartition` and then sorting within each partition because it can push the sorting down into the shuffle machinery. |

#### Action算子

下表列出了常见的action算子。

| Action                                             | Meaning                                                      |
| :------------------------------------------------- | :----------------------------------------------------------- |
| **reduce**(*func*)                                 | Aggregate the elements of the dataset using a function *func* (which takes two arguments and returns one). The function should be commutative and associative so that it can be computed correctly in parallel. |
| **collect**()                                      | Return all the elements of the dataset as an array at the driver program. This is usually useful after a filter or other operation that returns a sufficiently small subset of the data. |
| **count**()                                        | Return the number of elements in the dataset.                |
| **first**()                                        | Return the first element of the dataset (similar to take(1)). |
| **take**(*n*)                                      | Return an array with the first *n* elements of the dataset.  |
| **takeSample**(*withReplacement*, *num*, [*seed*]) | Return an array with a random sample of *num* elements of the dataset, with or without replacement, optionally pre-specifying a random number generator seed. |
| **takeOrdered**(*n*, *[ordering]*)                 | Return the first *n* elements of the RDD using either their natural order or a custom comparator. |
| **saveAsTextFile**(*path*)                         | Write the elements of the dataset as a text file (or set of text files) in a given directory in the local filesystem, HDFS or any other Hadoop-supported file system. Spark will call toString on each element to convert it to a line of text in the file. |
| **saveAsSequenceFile**(*path*) (Java and Scala)    | Write the elements of the dataset as a Hadoop SequenceFile in a given path in the local filesystem, HDFS or any other Hadoop-supported file system. This is available on RDDs of key-value pairs that implement Hadoop's Writable interface. In Scala, it is also available on types that are implicitly convertible to Writable (Spark includes conversions for basic types like Int, Double, String, etc). |
| **saveAsObjectFile**(*path*) (Java and Scala)      | Write the elements of the dataset in a simple format using Java serialization, which can then be loaded using `SparkContext.objectFile()`. |
| **countByKey**()                                   | Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key. |
| **foreach**(*func*)                                | Run a function *func* on each element of the dataset. This is usually done for side effects such as updating an [Accumulator](https://spark.apache.org/docs/3.0.2/rdd-programming-guide.html#accumulators) or interacting with external storage systems. **Note**: modifying variables other than Accumulators outside of the `foreach()` may result in undefined behavior. See [Understanding closures ](https://spark.apache.org/docs/3.0.2/rdd-programming-guide.html#understanding-closures-a-nameclosureslinka)for more details. |

#### Shuffle操作

`shuffle`是Spark重新分配数据的机制，以便在partition之间进行不同的分组。通常涉及跨executor、跨节点拷贝数据。涉及磁盘IO，网络IO，序列化，反序列化。

##### 背景

以`reduceByKey`为例，同一个key的数据需要放到同一个位置（我理解为同一个task）才能将此key的数据进行聚合。否则只能是局部的聚合。

在计算时，一个task处理一个partition，而当`reduceByKey`时，reduce task需要从所有partition中去读取每个key的所有值，然后在reduce task中进行聚合。

要使得shuffle之后的元素有序，可以使用如下3种方法

- `mapPartitions` 对每个partition进行排序
- `repartitionAndSortWithinPartitions`  通过重新分区对partition进行排序
- `sortBy`使得RDD全局有序

##### 性能影响

Shuffle是一项昂贵的操作，因为它涉及磁盘I/O、数据序列化和网络I/O。为了组织shuffle的数据，Spark生成了一组任务-映射任务来组织数据，以及一组reduce任务来聚合数据。该术语来自MapReduce，与Spark的映射和reduce操作没有直接关系。

map任务将数据保存在内存中，然后基于目标partition进行排序，并写入文件中。reduce任务从相应的executor上读取所需的文件。

shuffle操作会消耗大量的内存，因为会在内存中组织数据。当内存放不下shuffle的数据时，Spark会将数据溢写到磁盘，从而导致磁盘IO开销和GC的开销。

shuffle还会在磁盘上生成大量的中间文件。从Spark 1.3开始shuffle的中间文件会被保留（当重新计算时，就不用重新创建这些中间文件了），直到此RDD不再被引用之后，会被垃圾回收掉。

`spark.local.dir`配置参数可以决定中间文件的临时存储目录位置。

### RDD持久化

Spark可以在持久化（或缓存）数据。当持久化RDD时，每个节点都将计算的partitions存储在内存/磁盘中，方便在后序的操作中复用。缓存在迭代算法的关键工具。

使用`psersits()`或`cache()`方法将RDD标记为需要持久化，可以在第一次计算此RDD后（`action算子`），将此RDD保存在节点的内容/磁盘中。Spark的缓存是容错的（如果RDD某些partition丢失了，会自动进行重新计算）。

每个RDD可以选择不用的存储级别进行存储，cache()方法表示存储级别为`StorageLevel.MEMORY_ONLY`。通过指定不同的`StorageLevel`对象来设置存储级别。

如下表所示：

| Storage Level                          | Meaning                                                      |
| :------------------------------------- | :----------------------------------------------------------- |
| MEMORY_ONLY                            | 将RDD作为反序列化的Java对象存储在JVM中。如果内存存不下此RDD，则某些partition将不会被缓存，而是在每次需要时立即重新计算。这是默认级别。 |
| MEMORY_AND_DISK                        | 将RDD作为反序列化的Java对象存储在JVM中。如果内存存不下此RDD，将这些partition存储在磁盘上，并在需要时从那里读取。 |
| MEMORY_ONLY_SER (Java and Scala)       | 将RDD作为序列化的Java对象存储（每个partition为一个字节数组）。这样可以节省更多的空间，但读取时需要反序列化，会占用更多的CPU |
| MEMORY_AND_DISK_SER (Java and Scala)   | 类似于MEMORY_ONLY_SER，但将不能存到内存的partition溢出到磁盘，而不是在每次需要时立即重新计算。 |
| DISK_ONLY                              | 仅在磁盘上存储RDD分区。                                      |
| MEMORY_ONLY_2, MEMORY_AND_DISK_2, etc. | 与上面的级别相同，但在两个集群节点上复制每个分区。？？？     |
| OFF_HEAP (experimental)                | 类似于MEMORY_ONLY_SER，但将数据存储在堆外。                  |

注意：在Python中，存储的对象将始终使用Pickle库进行序列化，因此是否选择序列化级别无关紧要。Python中可用的存储级别包括MEMORY_ONLY、MEMORY_ONLY_2、MEMORY_AND_DISK、MEMORY-AND_DISK_2、DISK_ONLY和DISK_ONLY_2。

在shuffle过程中，为了避免节点挂掉之后，要重新计算整个RDD，Spark会自动持久化一部分的数据。

#### 如何选择存储级别

Spark的存储级别旨在提供内存使用和CPU效率之间的不同权衡。我们建议通过以下过程选择一个：

- 如果RDD与默认存储级别（MEMORY_ONLY）匹配得很好（内存存的下），请让它们保持原样。这是CPU效率最高的选项，允许RDD上的操作尽可能快地运行。
- 如果内存存不下，请尝试使用MEMORY_ONLY_SER并选择一个快速序列化库，以使对象的空间效率更高，但访问速度仍然相当快。（Java和Scala）
- 除非计算数据集的函数很昂贵，或者它们过滤了大量数据，否则不要溢出到磁盘。因为重新计算partition可能与从磁盘读取partition一样快。
- 如果您希望快速故障恢复，请使用复制的存储级别。所有存储级别都通过重新计算丢失的数据来提供完全的容错能力，但复制的存储级别允许您继续在RDD上运行任务，而无需等待重新计算丢失分区。

#### 移除数据

Spark自动监控每个节点上的缓存使用情况，并以`最近最少使用（LRU）`的方式丢弃旧数据分区。如果想要手动删除RDD，而不是等待它从缓存中掉出来，请使用`RDD.impossit()`方法。请注意，默认情况下，此方法不会阻塞。通过blocking=true参数，可以阻塞此RDD，直到释放此RDD资源后在删除此RDD数据。

### 共享变量

通常，Spark应用程序中的函数在集群的worker节点中执行时，函数中的所有变量在每个worker节点上是单独的副本，相互不影响。worker节点中变量的变化不会影响到driver中该变量的值。而跨task来读取共享变量的效率很低。Spark提供了两种共享变量来满足此目的。

#### 广播变量，broadcat variable

广播变量允许程序员在每个worker节点上缓存一个只读变量，而不是将此变量的副本和task一起发送。（如果是随任务发送的话，每个task保留一份变量副本，而广播变量是每个worker节点保留一份变量副本？？TODO）。通常是给每个节点广播一份数据集副本（不搬不大于10G吧）。节点上的task复用此数据集副本。

Spark application分为多个job，job可以分为多个stage，stage是一些列相关的task。stage之前都是被shuffle操作所分隔。Spark自动广播每个阶段中任务所需的公共数据。以这种方式广播的数据以序列化形式缓存，并在运行每个任务之前进行反序列化。这意味着，只有当跨多个阶段的任务需要相同的数据或以反序列化形式缓存数据很重要时，显式创建广播变量才有用。？？？

广播变量是通过调用SparkContext.broadcast(v)从变量v创建的。广播变量是v的包装器，可以通过调用value方法访问其值。

Scala

```scala
scala> val broadcastVar = sc.broadcast(Array(1, 2, 3))
broadcastVar: org.apache.spark.broadcast.Broadcast[Array[Int]] = Broadcast(0)

scala> broadcastVar.value
res0: Array[Int] = Array(1, 2, 3)
```

Java

```java
Broadcast<int[]> broadcastVar = sc.broadcast(new int[] {1, 2, 3});

broadcastVar.value();
// returns [1, 2, 3]
```



Python

```python
>>> broadcastVar = sc.broadcast([1, 2, 3])
<pyspark.broadcast.Broadcast object at 0x102789f10>

>>> broadcastVar.value
[1, 2, 3]
```

当基于变量v创建了广播变量broadcastVar之后，在Spark应用程序中的函数中都应该使用此广播变量broadcastVar，而不是使用变量v。这样v就不用被传递多次（数据已经放到广播变量里了，会自动广播了）；而且变量v在广播之后不能被修改，否则会造成不同节点的广播变量的值不一样。

使用`broadcastVar.unpersist()`方法可以释放拷贝到executor的广播变量。如果后续又要使用此广播变量，则需要再重新广播。

使用`broadcastVar.destory()`可以永久释放广播变量。后续无法再使用此广播变量了。

这两个方法默认是不阻塞的，要阻塞使用参数blocking=true。

#### 累加器，accumulator

累加器是一种只能"追加"的变量。累加器可以同来实现计数器（counter）或求和（sum）。Spark原生支持的累加器是数字类型的，如果要支持其他类型，程序员需要自己实现。

可以创建命名的/未命名的累加器。如下图所示，可以在job的Spark web ui界面查看到累加器的值

![Accumulators in the Spark UI](Spark.assets/spark-webui-accumulators.png)

Scala

创建数字类型的累加器：

- `SparkContext.longAccumulator()` Long型的累加器
- `SparkContext.doubleAccumulator()` Double类型的累加器

task在运行过程中，可以调用累加器的`add()`方法来追加。

<font color="red">**task不能读取到累加器的值，只有driver才能获取到累加器的值。**</font>

```scala
scala> val accum = sc.longAccumulator("My Accumulator")
accum: org.apache.spark.util.LongAccumulator = LongAccumulator(id: 0, name: Some(My Accumulator), value: 0)

scala> sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum.add(x))
...
10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s

scala> accum.value
res2: Long = 10
```

创建自定义的累加器需要继承`AccumulatorV2`抽象类。

继承`AccumulatorV2`抽象类之后，必须要重写（override）一下几个方法：

- `reset` 用于将累加器重置为零。
- `add` 用于向累加其中添加一个元素。
- `merge` 用于将另一个相同类型的累加器合并到当前累加器。
- 其他需要重写的方法，https://spark.apache.org/docs/3.0.2/api/scala/org/apache/spark/util/AccumulatorV2.html



```scala
class VectorAccumulatorV2 extends AccumulatorV2[MyVector, MyVector] {

  private val myVector: MyVector = MyVector.createZeroVector

  def reset(): Unit = {
    myVector.reset()
  }

  def add(v: MyVector): Unit = {
    myVector.add(v)
  }
  ...
}

// Then, create an Accumulator of this type:
val myVectorAcc = new VectorAccumulatorV2
// Then, register it into spark context:
sc.register(myVectorAcc, "MyVectorAcc1")
```

累加器的更新也是惰性的，它们只会在RDD执行到action算子时才会真正地被更新。

```scala
val accum = sc.longAccumulator
data.map { x => accum.add(x); x }
// Here, accum is still 0 because no actions have caused the map operation to be computed.
```



```java
LongAccumulator accum = jsc.sc().longAccumulator();

sc.parallelize(Arrays.asList(1, 2, 3, 4)).foreach(x -> accum.add(x));
// ...
// 10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s

accum.value();
// returns 10
```



Python

```python
>>> accum = sc.accumulator(0)
>>> accum
Accumulator<id=0, value=0>

>>> sc.parallelize([1, 2, 3, 4]).foreach(lambda x: accum.add(x))
...
10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s

>>> accum.value
10
```



### 将Spark应用程序部署到集群中运行

Java/Scala程序打包成JAR，Python则为.py或.zip文件，然后是通`bin/spark-submit`命令来提交应用程序到集群去执行。

参考：https://spark.apache.org/docs/3.0.2/submitting-applications.html

### 通过Java/Scala运行Spark jobs

org.apache.spark.launcher包提供了使用简单Java API将spark作业作为子进程启动。

### 单元测试

单元测试时，通常将master设置为local，然后创建`SparkContext`并运行spark应用程序，然后通过`SparkContext.stop()`停止。通常将`SparkContext.stop()`放在finally代码块中，或者`tearDown`方法中。



## 1.4 Spark SQL, DataFrame和Dataset

Spark SQL是Spark中用来处理结构化数据的模块。使用SQL语句或DataSet的API都可以与Spark SQL交互。

### SQL

Spark SQL可以用来执行SQL查询。Spark SQL支持从Hive中读取库表数据。

配置Spark SQL 可以操作Hive表中的数据，https://spark.apache.org/docs/3.0.2/sql-data-sources-hive-tables.html

Spark SQL的编程语言API返回的结果类型为DataSet或DataFrame。

可以通过命令行（如`spark-sql`）访问Spark SQL，也可以通过编程的方式通过JDBC/ODBC与Spark SQL进行交互。

### DataSet和DataFrame

DataSet就是一种分布式数据集合。DataSet是Spark 1.6时增加的一个接口，它具有RDD的优点（强类型、支持lambda函数）和Spark SQL优化执行引擎的优点。基于JVM对象，利用transformation算子（如map、flatMap、filter等）就可以构造出一个DataSet对象了。

DataSet API目前（Spark 3.3.X）只支持Scala和Java。Python和R中不支持。



DataFrame

**Dataframe是Dataset的特例**，`DataFrame`=`Dataset[Row]` ，所以可以通过as方法将`Dataframe`转换为Dataset。Row是一个类型，跟Car、Person这些的类型一样，所有的表结构信息我都用Row来表示。

可以根据结构化数据文件、Hive中的表、外部数据库、已有的RDD对象等来构建出`DataFrame`对象。

`DataFrame API` 在Java、Scala、Python、R中均可用。在Java和Scala中DataFrame就是Row对象的DataSet。Scala中`Dataset[Row]`，Java中`Dataset<Row>`



## 1.5 Structured Streaming

## 1.6 Spark Streaming (DStreams)

TODO

## 1.7 Mlib (Machine Learning)

TODO

## 1.8 GraphX (Graph Processing)

TODO

## 1.9 SparkR (R on Spark)

TODO



## 1.10 集群模式（Cluster Mode）

Spark应用程序其实就是运行在集群中的一系列独立的进程，这些进程由Driver进程中的SparkContext对象来协调。

SparkContext可以连接到多种集群管理器，Standalone、Mesos、YARN、Kubernetes。

1. SparkContext连接上集群管理器
2. SparkContext发送应用程序代码（通常为JAR或Python文件）给executor
3. SparkContext发送task给executor，executor运行task。

![Spark cluster components](Spark.assets/cluster-overview.png)

需要注意的是：

- 每个应用程序都属于自己的executor进程。每个应用程序的executor运行在不同的jvm中，每个executor可以通过多线程运行多个task。如果不将数据写入外部存储系统，则无法在不同的Spark应用程序（SparkContext实例）之间共享数据。
- 在此应用程序的生命周期内，Driver程序必须要监听和接收executor的连接。
- 因为Driver进程需要调度task到executor上，因此Driver进程最好与worker节点距离近。一般在同一局域网中。

### 集群管理器分类

截止到Spark 3.3.X，目前支持如下几种集群管理器。

- [Standalone](https://spark.apache.org/docs/3.0.2/spark-standalone.html) Spark自带的一个简单的集群管理器，配置使用较简单。
- [Mesos](https://spark.apache.org/docs/3.0.2/running-on-mesos.html) 一个通用的集群管理器，也可以运行Hadoop MapReduce
- [YARN](https://spark.apache.org/docs/3.0.2/running-on-yarn.html) Hadoop2中的资源管理器
- [Kubernetes](https://spark.apache.org/docs/3.0.2/running-on-kubernetes.html) 一个用于自动容器化应用程序的部署、扩展和管理的开源系统。

#### Spark Standalone模式

参考：https://spark.apache.org/docs/3.0.2/spark-standalone.html

##### 在集群中安装Spark Standalone

将编译好的Spark放置到集群中的每个节点就行了。

##### 手动启动Spark Standalone集群

启动一个standalone的master服务。

```shell
./sbin/start-master.sh
```

启动成功后，会在控制台打印出`spark://HOST:PORT`，这就是mater的URL，也可以在`http://localhost:8080`（默认）查看master的URL。在创建SparkContext时，传递此master的URL给SparkContext。

启动一个或多个worker服务，并让worker连接到master。

```shell
./sbin/start-slave.sh <master-spark-URL>
```

worker启动之后，可以在master的Spark web ui `http://localhost:8080`（默认）中查看有哪些worker。

启动master和worker时支持的配置项：

| Argument                    | Meaning                                                      |
| :-------------------------- | :----------------------------------------------------------- |
| `-h HOST`, `--host HOST`    | Hostname to listen on                                        |
| `-i HOST`, `--ip HOST`      | Hostname to listen on (**deprecated**, use -h or --host)     |
| `-p PORT`, `--port PORT`    | Port for service to listen on (default: 7077 for master, random for worker) |
| `--webui-port PORT`         | Port for web UI (default: 8080 for master, 8081 for worker)  |
| `-c CORES`, `--cores CORES` | Total CPU cores to allow Spark applications to use on the machine (default: all available); **only on worker** |
| `-m MEM`, `--memory MEM`    | Total amount of memory to allow Spark applications to use on the machine, in a format like 1000M or 2G (default: your machine's total RAM minus 1 GiB); **only on worker** |
| `-d DIR`, `--work-dir DIR`  | Directory to use for scratch space and job output logs (default: SPARK_HOME/work); **only on worker** |
| `--properties-file FILE`    | Path to a custom Spark properties file to load (default: conf/spark-defaults.conf) |

在使用集群启停脚本之前，需要一个`${SPARK_HOME}/conf/salves`文件，表示每行一个worker的主机名，我们需要启停这些worker。`conf/slaves`文件不存在时，则只在本机启动一个worker，主要用于测试。

master通过`ssh`去访问每个worker。可以配置master免密访问所有worker，也可以在环境变量中配置`SPARK_SSH_FOREGROUN`来让master可以根据账/密来访问每个worker。

下述脚本在`${SPARK_HOME}`目录下。

- `sbin/start-master.sh` - 在本机启动一个master服务。
- `sbin/start-slaves.sh` - 在`conf/slaves`中的机器上分别启动一个worker服务。
- `sbin/start-slave.sh` - 在本机启动一个worker服务。
- `sbin/start-all.sh` - 在本机启动master服务，根据`conf/slaves`中的机器上启动worker服务。
- `sbin/stop-master.sh` - 停止已启动的master服务。TODO不确定是否一定要去master所在机器执行。
- `sbin/stop-slave.sh` - 停止本机上的worker服务。
- `sbin/stop-slaves.sh` - 停止`conf/slaves`中机器上的worker服务。
- `sbin/stop-all.sh` - 停止所有master和worker服务。

**TODO某些脚本必须在运行Spark Standalone的master服务的机器上执行。**

可以通过`${SPARK_HOME}/conf/spark-env.sh`中配置环境变量来进一步配置Standalone集群。

具体配置方法参考：https://spark.apache.org/docs/3.0.2/spark-standalone.html

***注意：目前，集群启停脚本在windows系统中是不支持的。要在windows系统中运行Spark Standalone集群，需要手动启动master和worker服务。***

##### 资源分配和配置

分为两部分：配置worker的资源和给应用程序分配的资源。

 `spark.worker.resource.{resourceName}.amount` 控制分配给每个worker的资源。用户还必须指定`spark.worker.resourcesFile`或`spark.workr.resource.{resourceName}.discoveryScript`。

当提交哒Spark Standalone的应用程序使用的是client模式时，需要通过`spark.Driver.resourcesFile`或`spark.driveer.resource.{resourceName}.discoveryScript`来指定driver所使用的资源。

运行交互式`spoark-shell`

```shell
./bin/spark-shell --master spark://IP:PORT
```

在Spark Standalone集群中运行应用程序

```scala
val conf = new SparkConf()
  .setMaster("spark://IP:PORT")
  .setAppName("application11")
val sc = new SparkContext(conf
```

##### 运行Spark应用程序

Spark Standalone有两种部署模式：client和cluster。

- client模式。driver运行在`spark-submit`这个进程中，提交完应用程序，这个进程仍然不会退出，因为driver还需要。
- cluster模式。driver运行在集群中的某个worker节点的进程中，`spark-submit`的进程在提交完应用程序之后就退出来。

Spark Standalone使用`--supervise`可以自动重启退出状态码为非零的应用程序。使用刚如下命令可以kill某应用程序。

```shell
./bin/spark-class org.apache.spark.deploy.Client kill <master url> <driver ID>
```

`http://<master url>:8080`可以查看到driver的id是多少。

##### 资源调度

目前（Spark 3.3.X），Spark Standalone支持持FIFO调度。（我理解所有应用程序都在一个队列中运行，不像YARN有资源和租户隔离）。

通过 `spark.cores.max`可以控制应用程序所用的core数。

通过配置master进程中的`SPARK_MASTER_OPTS`参数，可以设置应用程序默认使用的core数。

```bash
export SPARK_MASTER_OPTS="-Dspark.deploy.defaultCores=<value>"
```



##### executor调度

`spark.executor.cores`可控制每个executor的core数量。

##### 监控和日志

master和worker服务都有自己的web ui，展示集群和job的统计信息。

每个job的详细日志写入了每个工作节点的工作目录（默认为`${SPARK_HOME}/work`）。每个job会有两个目录：`stdout`和`stderr`

##### 和Hadoop一起运行

在集群内的节点上都部署上Spark，然后作为单独的服务来运行，Spark服务可以通过`hdfs://<namenode>:9000/<path>`来方法hdfs。

也可以在局域网内，单独搞几个节点专门安装Spark，运行Spark Standalone集群。

##### 配置端口已保证网络安全

通常而言，Spark集群不会部署在公网，而是部署在企业内部的局域网中。通过防火墙、安全组更配置可以限制哪些主机可以访问集群，保证集群的网络安全。

##### 高可用

Spark Standalone集群可以识别出故障的worker，然后将job转移到其他worker。前面配置的都是单点的master，存在单点故障，需要实现高可用来解决单点故障问题。

###### 通过Zookeeper实现Standby Master

利用ZK提供的leader选举和状态存储功能，我们就可以在集群中运行多个master，这些master需要连接到同一个ZK集群。利用ZK的leader选举，一个master为active，其他master为standby。当active挂了，之后standby进行选举，选举出新的active，并恢复前active的状态，保证状态数据一致。

ha的故障恢复过程可能要花费1-2分钟，已经提价的应用程序不影响的，新提交的应用程序在这1-2分钟，应该是提不了的。

要启用恢复模式，在`spark-env.sh`中使用如下参数来配置`SPARK_DAEMON_JAVA_OPTS`：

| Property Name                | Default | Meaning                                                      | Since Version |
| :--------------------------- | :------ | :----------------------------------------------------------- | :------------ |
| `spark.deploy.recoveryMode`  | NONE    | The recovery mode setting to recover submitted Spark jobs with cluster mode when it failed and relaunches. This is only applicable for cluster mode when running with Standalone or Mesos. | 0.8.1         |
| `spark.deploy.zookeeper.url` | None    | When `spark.deploy.recoveryMode` is set to ZOOKEEPER, this configuration is used to set the zookeeper URL to connect to. | 0.8.1         |
| `spark.deploy.zookeeper.dir` | None    | When `spark.deploy.recoveryMode` is set to ZOOKEEPER, this configuration is used to set the zookeeper directory to store recovery state. | 0.8.1         |

配置好高可用之后，Spark的客户端（如`spark-shell`或`SparkContext`）需要指定amster 的URL为此格式： `spark://host1:port1,host2:port2`。依次尝试与master建立连接，直到遇到active的master。

###### 通过本地文件系统实现单节点故障恢复

ZooKeeper是实现生产级高可用性的最佳方法，但如果你只想在master进程停止时重新启动它，FILESYSTEM模式可以处理它。当应用程序和Workers注册时，它们有足够的状态写入到提供的目录中，以便在master进程重新启动时恢复它们。

要启用FILESYSTEM的恢复模式，在`spark-env.sh`中使用如下参数来配置`SPARK_DAEMON_JAVA_OPTS`：

| System property                  | Meaning                                                      | Since Version |
| :------------------------------- | :----------------------------------------------------------- | :------------ |
| `spark.deploy.recoveryMode`      | Set to FILESYSTEM to enable single-node recovery mode (default: NONE). | 0.8.1         |
| `spark.deploy.recoveryDirectory` | The directory in which Spark will store recovery state, accessible from the Master's perspective. | 0.8.1         |

#### Spark on Mesos模式

TODO

#### Spark on YARN模式

参考：https://spark.apache.org/docs/3.0.2/running-on-yarn.html

从Spark 0.6.0开始，支持YARN作为集群管理器了。

##### 运行Spark on YARN

确保`HADOOP_CONF_DIR`或`YARN_CONF_DIR`指向包含HADOOP集群（客户端）配置文件的目录。这些配置在Spar写入HDFS，并连接到YARN ResourceManager时会用到。此目录中包含的配置将分发到YARN集群，以便应用程序使用的所有容器都使用相同的配置。如果引用了非YARN的配置文件，那么这些配置应该在Spark 应用程序的配置文件中配置好。（当以client模式部署时，driver、executor和AM的一些配置）。

Spark on YARN有两种部署模式：cluster和client。

- cluster。Spark的driver运行在application master（AM）中，由YARN所管理。`spark-submit`作为YARN的客户端提交完应用程序之后就结束了。

- client。Spark的driver运行在`spark-submit`同一个进程中，而AM只是用来向YARN请求资源的。

  使用Spark on YARN，`--master yarn`而不用指定具体的master的URL，因为YARN的Resource Manager的地址是从Hadoop的配置文件中获取。

以cluster模式运行一个Spark应用程序的格式应如下：

```shell
$ ./bin/spark-submit \
--class path.to.your.Class \
--master yarn \
--deploy-mode cluster \
[options] \
<app jar> \
[app options]
```

例如：

```shell
$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode cluster \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    --queue thequeue \
    examples/jars/spark-examples*.jar \
    10
```

`spark-submit`会启动一个YARN的客户端，向YARN提交作业。提交之后YARN会分配一个container用来运行AM。而SparkPi程序作为AM的子线程运行在其中。

client模式则将`--deploy-mode`设置为`client`即可。

##### 准备

https://blog.csdn.net/penriver/article/details/116158249

为了保证Spark能够获取到其运行时所需的jar包，我们需要设置`spark.yarn.archive`或`spark.yarn.jars`。如果两个都配置`spark.yarn.archive`的优先级更高。

如果这两个参数都没有指定，则Spark会把`${SPARK_HOME}/jars`下的所有jar包打成个`.zip`包，并上传到HDFS中，这个过程非常耗时。

参考：https://spark.apache.org/docs/3.0.2/configuration.html

##### debugging你的应用程序

在YARN中，executor和AM都是运行在container中的。YARN有两种处理container日志的方式：

- 开启日志聚合。

  如果开启了日志聚合(`yarn.log-aggregation-enable`设置为true)，container的日志会被复制到HDFS，并且从worker节点本地中删除这些日志文件。

  方式一、通过`yarn logs --applicationId <应用程序ID> >> /tmp/<应用程序ID>.log`可以查看应用程序的所有日志了。

  方式二、通过查看YARN配置中的`yarn.nodemanager.remote-app-log-dir`和`yarn.nodemanager.remote-app-log-dir-suffix`可知道日志在HDFS中的哪个路径下。

  方式三、通过Spark Web UI在的Exectuors页面查看。需要同时运行`Spark历史服务器`和`MapReduce历史服务器`。需要配置好`yarn-site.xml`中的`yarn.log.server.url`配置项。`Spark历史服务器`Web UI上的日志URL会重定向到`MapReduce历史服务器`，进而可以查看聚合之后的日志。

- 未开启日志聚合。

  若没开启日志聚合，则应用程序的日志会保留在执行作业的节点的本地目录（`YARN_APP_LOGS_DIR`）中。`YARN_APP_LOGS_DIR`的子目录按照`application ID`和container ID`来组织日志文件。

  方式一、可以登录对应的节点查看每个container的日志。

  方式二、Spark Web UI中的Executors页面查看应用程序的日志（不需要运行`MapReduce历史服务器`）。

  `yarn.nodemanager.local-dir`目录中存放了应用存储缓存在本地目录中的一些文件。如启动脚本、JAR包、用于启动每个container的所有环境变量。

我们还可以使用自定义的`log4j`配置项来修改AM或executor的日志记录格式：

- 使用`spark-submit --files log4j.properties`，随应用程序上传时，上传自定义的`log4j.properties`文件。
- 添加配置项`-Dlog4j.configuration=<日志配置文件位置>`到`spark.driver.extraJavaOptions`或`spark.executor.extraJavaOptions`，此日志配置文件必须在所有节点都存在。
- 修改`${SPARK_CONF_DIR}/log4j.properties`文件，会自动被上传

YARN 3.1.0中增加了YARN上的资源调度。理想情况下，资源被隔离设置，以便执行者只能看到分配的资源。如果未启用隔离，则用户负责创建一个发现脚本，以确保执行者之间不共享资源。



##### 注意事项

- Whether core requests are honored in scheduling decisions depends on which scheduler is in use and how it is configured.
- 在cluster模式下，`Spark executor`和`Spark driver`使用的本地目录是YARN配置的 `YARN.nodemanager.local dirs`。cluster模式会忽略`spark.local.dir`的配置。client模式下，Spark执行器将使用为YARN配置的本地目录，而`Spark driver`将使用`spark.local.dir`的配置。这是因为在client模式中，`driver`不会在YARN集群上运行，只有`executor`会运行在YARN集群中。
- The `--files` and `--archives` options support specifying file names with the # similar to Hadoop. For example, you can specify: `--files localtest.txt#appSees.txt` and this will upload the file you have locally named `localtest.txt` into HDFS but this will be linked to by the name `appSees.txt`, and your application should use the name as `appSees.txt` to reference it when running on YARN.
- The `--jars` option allows the `SparkContext.addJar` function to work if you are using it with local files and running in `cluster` mode. It does not need to be used if you are using it with HDFS, HTTP, HTTPS, or FTP files.
- 4

##### Keberos

参考：https://spark.apache.org/docs/3.0.2/security.html#kerberos

Spark on YARN的keberos相关配置。

| Property Name                        | Default | Meaning                                                      | Since Version |
| :----------------------------------- | :------ | :----------------------------------------------------------- | :------------ |
| `spark.kerberos.keytab`              | (none)  | The full path to the file that contains the keytab for the principal specified above. This keytab will be copied to the node running the YARN Application Master via the YARN Distributed Cache, and will be used for renewing the login tickets and the delegation tokens periodically. Equivalent to the `--keytab` command line argument. (Works also with the "local" master.) | 3.0.0         |
| `spark.kerberos.principal`           | (none)  | Principal to be used to login to KDC, while running on secure clusters. Equivalent to the `--principal` command line argument. (Works also with the "local" master.) | 3.0.0         |
| `spark.yarn.kerberos.relogin.period` | 1m      | How often to check whether the kerberos TGT should be renewed. This should be set to a value that is shorter than the TGT renewal period (or the TGT lifetime if TGT renewal is not enabled). The default value should be enough for most deployments. | 2.3           |

##### 配置外部shuffle服务（External Shuffle Service）

如何在YARN集群中的每个NodeManager张都启动`Spark Shuffle Service`呢？

1. 编译Spark，使用预编译好的Spark可跳过此步骤
2. 找到`spark-<version>-yarn-shuffle.jar`。如果您自己构建spark，它应该位于`$spark_HOME/common/networkyarn/target/scala-<version>`之下，如果您使用的是发行版，它应该在`yarn目录`TODO之下。
3. 将此jar添加到YARN集群中所有NodeManager的classpath中。
4. 在所有节点的`yarn-site.xml`文件中添加`spark_shuffle=yarn.nodemanager.aux-services`，添加`yarn.nodemanager.aux-services.spark_shuffle.clas=org.apache.spark.network.yarn.YarnShuffleService`
5. 增大NodeManager的堆大小，避免shuffle期间频繁GC，修改`etc/hadoop/yarn-env.sh`文件中的`YARN_HEAPSIZE`配置项。
6. 重启集群中的所有NodeManager。

`Spark Shuffle Servie`还有如下额外配置。



| Property Name                      | Default | Meaning                                                      |
| :--------------------------------- | :------ | :----------------------------------------------------------- |
| `spark.yarn.shuffle.stopOnFailure` | `false` | 当初始化Spark Shuffle服务失败时，是否停止此`NodeManager`。此配置可以避免因Spark Shuffle服务未运行而导致应用程序失败。 |

##### 使用Oozie运行你的应用程序

`Apache Oozie`支持以工作流（workflow）的形式运行Spark 应用程序。

##### 使用Spark历史服务器，而不是Spark Web UI

可以设置`Spark历史服务器`作为应用程序的跟踪URL（还记得YARN界面跳转到哪儿吗，就是这个）。

- 在应用程序侧，在spark的配置中设置`spark.yarn.historyServer.allowTracking=true`。如果application的UI被禁用，Spark便会使用`Spark历史服务器`的URL作为跟踪URL。
- 在`Spark历史服务器`上，将`org.apache.Spark.deploy.yarn.YarnProxyRedirectFilter`添加到`spark.ui.filter`配置中的列表中。 

####  Spark on Kubernetes模式

TODO

### 提交应用

使用`${SPARK_HOME}/bin/spark-submit`脚本来提交应用程序

参考：https://spark.apache.org/docs/3.0.2/submitting-applications.html

#### 绑定应用程序的依赖项

如果您的代码依赖于其他项目，则需要将它们与应用程序一起打包，以便将代码分发到Spark集群。为此，需要创建一个包含代码及其依赖项的jar（或“uber”jar）。sbt和Maven都有相应插件。创建jar时，将Spark和Hadoop的依赖级别设置为`proviede`，因为它们是由集群管理器在运行时提供的，不需要打包进jar中。

如果应用程序是由Python编写的，可以使用`--py-files`参数指定需要提交的`.py`或`.zip`或`.egg`文件。

#### 使用`spark-submit`运行应用程序



```shell
./bin/spark-submit \
  --class <main-class> \
  --master <master-url> \
  --deploy-mode <deploy-mode> \
  --conf <key>=<value> \
  ... # other options
  <application-jar> \
  [application-arguments]
```

其中的`<application-jar>` 表示我们要提交运行的jar包路径，支持`hdfs://path`或`file://path`。

`[application-arguments]` 表示传递给应用程序的主类的参数，多个参数空格分隔。

常用的选项：

- `--class` 应用程序的主类（应用程序的入口类），需要全路径。Java/Scala
- `--master` 集群的Master URL。
- `--deploy-mode` 默认为client。cluster/client。在本地运行driver，还是在集群中的某worker中运行。
- `--conf` 任意的Spark配置参数。格式为key=value。多个配置参数格式为`--conf<key>=<value>--conf<key 2>=<value 2>`。若value中含有空格，使用双引号包裹"key=value 1"
- `--jars` 需要添加到driver和executor的classpath中的jar包，逗号分隔。
- `--packages` 需要添加到driver和executor的classpath的maven依赖。逗号分隔。格式为`groupId:artifactId:version`。配置之后，会依次从本地maven仓库、maven中央仓库、`--repositories`配置的远程maven仓库中获取此依赖。
- `--py-files` 支持`.py`, `.zip`, `.egg`格式，需要添加到PYTHONPATH的文件，逗号分隔。
- `--files` 需要放到每个executor的工作目录的文件，逗号分隔。可以通过SparkFiles.get(fileName)获取这些文件。
- `--name` 默认为TODO。应用程序的名称
- `--driver-memory` 默认为1024M。dirver进程的内存大小，如1000M，2G。
- `--driver-cores` 默认为1。`--deploy-mode`=`cluster`时有效。
- `--executor-moemry` 默认为1G。executor的内存，如1000M，2G。
- `--num-executors` 默认为2。此应用程序总的executor的数量。如果启用了spark资源动态调度，则初始的executor数量>=此值。
- `--properties-file` 应用程序用到的配置文件，若未指定，则默认读取`${SPARK_HOME}/conf/spark-defaults.conf`中的配置项。
- 其他选项，`spark-submit --help`

当通过网关服务器（例如CDH的gateway，腾旭云的Router节点）来提交应用程序，那么可以使用`--deploy-mode`=`client`，因为driver和executor的距离很近。

但是，如果提交应用程序的机器与集群距离很远，那么最好使用`--deploy-mode=client`，这样可以大大减小driver和executor通信时的网络延迟和开销。

目前（Spark 3.3.X），针对Python应用程序standalone模式不支持`--deploy-mode`=`cluster`。

要提交Python应用程序，只需在`<application-jar>`出替换为你的py文件即可。`--py-files` 提供此py文件所用到的python包或者模块。

`spark-submit`使用示例：

```shell
# Run application locally on 8 cores
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local[8] \
  /path/to/examples.jar \
  100

# Run on a Spark standalone cluster in client deploy mode
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# Run on a Spark standalone cluster in cluster deploy mode with supervise
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --deploy-mode cluster \
  --supervise \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# Run on a YARN cluster
export HADOOP_CONF_DIR=XXX
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn \
  --deploy-mode cluster \  # can be client for client mode
  --executor-memory 20G \
  --num-executors 50 \
  /path/to/examples.jar \
  1000

# Run a Python application on a Spark standalone cluster
./bin/spark-submit \
  --master spark://207.184.161.138:7077 \
  examples/src/main/python/pi.py \
  1000

# Run on a Mesos cluster in cluster deploy mode with supervise
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master mesos://207.184.161.138:7077 \
  --deploy-mode cluster \
  --supervise \
  --executor-memory 20G \
  --total-executor-cores 100 \
  http://path/to/examples.jar \
  1000

# Run on a Kubernetes cluster in cluster deploy mode
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master k8s://xx.yy.zz.ww:443 \
  --deploy-mode cluster \
  --executor-memory 20G \
  --num-executors 50 \
  http://path/to/examples.jar \
  1000
```



#### Master URLs

目前Spark支持如下几种Master URL：

| Master URL                        | Meaning                                                      |
| :-------------------------------- | :----------------------------------------------------------- |
| `local                            | 在本地运行Spark，单线程，无并发。                            |
| `local[K]`                        | 在本地运行Spark，K个线程                                     |
| `local[K,F]`                      | Run Spark locally with K worker threads and F maxFailures (see [spark.task.maxFailures](https://spark.apache.org/docs/3.0.2/configuration.html#scheduling) for an explanation of this variable) |
| `local[*]`                        | 在本地运行Spark，线程数为cpu核数。                           |
| `local[*,F]`                      | Run Spark locally with as many worker threads as logical cores on your machine and F maxFailures. |
| `spark://HOST:PORT`               | 连接到指定的[Spark standalone cluster](https://spark.apache.org/docs/3.0.2/spark-standalone.html) |
| `spark://HOST1:PORT1,HOST2:PORT2` | 连接到指定的 [Spark standalone cluster with standby masters with Zookeeper](https://spark.apache.org/docs/3.0.2/spark-standalone.html#standby-masters-with-zookeeper). 需要列出所有master |
| `mesos://HOST:PORT`               | 连接到Mesos集群。若Mesos集群使用了Zookeeper，则使用`mesos://zk://...` |
| `yarn`                            | 连接到YARN集群                                               |
| `k8s://HOST:PORT`                 | 连接到Kubernetes集群，目前仅支持--deploy-mode=cluster。默认使用TLS加密，若要不使用加密，则使用`k8s://http://HOST:PORT`。 |

#### 从文件中加载配置

`spark-submit`命令可以从文件中读取Spark配置项，并传递给你的应用程序。默认从`${SPARK_HOME}/conf/spark-defaults.conf`读取。

参考：https://spark.apache.org/docs/3.0.2/configuration.html#loading-default-configurations



#### 高级依赖管理

`spark-submit`命令行选项中的`--jars`

- `file://` 每个executor会从driver的http文件服务器拉取JARS
- `hdfs://`,`http://`,`https://`,`ftp://` 从此URL拉取JARS
- `local://` 从每个worker节点的本地获取JARS

### 监控

每个驱动程序都有一个web UI，通常在端口4040上，显示该job正在运行的task、executor和storage使用情况的信息。只需在web浏览器中转到`http://<driver node>:4040`即可访问此UI。

参考：https://spark.apache.org/docs/3.0.2/monitoring.html

### job调度

Spark可以在应用程序之间（在集群管理器级别）和应用程序内部（如果在同一SparkContext上进行多个计算）控制资源分配。

参考：https://spark.apache.org/docs/3.0.2/job-scheduling.html

### 术语表

https://blog.csdn.net/u013603364/article/details/124207781

| Term                           | Meaning                                                      |
| :----------------------------- | :----------------------------------------------------------- |
| Application                    | 基于Spark构建的用户程序。由集群上的*驱动程序*和*执行器*组成。Driver和Executor |
| Application jar                | 包含用户的应用程序的JAR包，在某些情况下，用户会希望创建一个包含其应用程序及其依赖项的“uber jar”。用户的jar不应包含Hadoop或Spark库，这些库将在运行时从集群中获取。因为把这些包都加上的话，打的jar包会比较大。 |
| Driver program                 | 运行应用程序中main()方法的进程，此进程会创建一个SparkContext对象。 |
| Cluster manager                | 指的是在集群上获取资源的外部服务，常用的有 (e.g. standalone manager, Mesos, YARN) |
| Deploy mode                    | cluster模式下，在集群内部运行Driver；client模式下，在集群外部运行client。以YARN为例，cluster模式会启动一个容器来运行Driver（对应YARN中的AM），client模式会在提交应用程序的节点启动一个进程来运行Driver。 |
| Worker node                    | 集群中的计算节点，用于执行应用程序的代码。类似于Yarn中的NodeManager节点。 |
| Executor                       | Application运行在Worker节点上的一个进程，该进程负责运行Task，并且负责将数据存在内存或者磁盘上， 每个Application都有各自独立的一批Executor |
| Task                           | 工作单元，SparkContext会将task发给对应的executor，由executor来执行这些task。一个partition需要一个task来处理。 |
| Job                            | 由一个或多个调度阶段Stage所组成的一次计算作业。job包含多个stage，stage包含多个task。 |
| Stage                          | job会被划分为更小的任务集，称之为stage，包含多个task。通常stage之间被spark shuffle所隔开。 |
| RDD：弹性分布式数据集          | Resillient Distributed Dataset，Spark的基本计算单元，可以通过一系列算子进行操作(主要有Transformation和Action操作)， |
| NarrowDependency窄依赖         | 子RDD的分区只依赖父RDD的一个分区                             |
| ShuffleDependency宽依赖        | 子RDD的分区只依赖父RDD的多个分区                             |
| DAG：有向无环图                | Directed Acycle graph，反应RDD之间的依赖关系，<br/>  DAG其实就是一个JOB(会根据依赖关系被划分成多个Stage)<br/>  注意:一个Spark程序会有1~n个DAG,调用一次Action就会形成一个DAG |
| DAGScheduler：有向无环图调度器 | 基于DAG划分Stage 并以TaskSet的形式提交Stage给TaskScheduler;<br/>  负责将作业拆分成不同阶段的具有依赖关系的多批任务;<br/>  最重要的任务之一就是：计算作业和任务的依赖关系，制定调度逻辑。<br/>  在SparkContext初始化的过程中被实例化，一个SparkContext对应创建一个DAGScheduler。 |
| TaskScheduler：任务调度器      | 将Taskset提交给worker(集群)运行并回报结果;<br/><br/>  负责每个具体任务的实际物理调度。 |
| SHS Web UI                     | Spark History Server                                         |



## 1.11 Spark配置项

参考：https://spark.apache.org/docs/3.0.2/configuration.html

TODO

Spark有三种类型的属性：

- Spark Properties

  此类属性用来控制application的行为，可以基于每个应用程序单独配置不同的属性值。

- 环境变量

- 日志配置

### Spark Properties

#### 动态加载Spark Properties

Spark shell和`spark-submit`有两种动态加载Spark Properties配置的方法：

- 命令行参数

  通过`--conf` 来指定spark的properties。

  ```scala
  ./bin/spark-submit --name "My app" --master local[4] --conf spark.eventLog.enabled=false
    --conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps" myApp.jar
  ```

  

- `conf/spark-defaults.conf`

  `spark-submit`会读取`conf/spark-defaults.conf`中的配置项，通过修改此配置中配置项的值。最终会与`SparkConf`中的配置进行合并，然后创建`SparkContext`实例对象。

写死在代码中的配置项的优先级最高，代码中的配置项 > 命令行中的配置项 > 配置文件中的配置项。

通过`http://<dirver>:4040`打开应用程序的Web UI，然后在"Environment"页面可以查看当前应用程序的spark properties。

***注意：只有通过`spark-defaults.con`、`SparkConf`或`命令行显式指定的配置项`才会显示出来。***

##### application相关配置

TODO

##### runtime相关配置

TODO

##### shuffle相关配置

| Property Name                                                | Default           | Meaning                                                      | Since Version |
| :----------------------------------------------------------- | :---------------- | :----------------------------------------------------------- | :------------ |
| <font color="red">`spark.reducer.maxSizeInFlight`</font>     | 48m               | 参数说明：该参数用于设置`shuffle read`任务的buff缓冲区大小，该缓冲区决定一次可以拉取多少数据。</br>调优建议：如果可用内存资源足够，则可以增加参数的大小（例如96m），从而减少拉取数据的次数，这可以减少网络传输的次数并提高性能。 在实践中发现，合理调整参数后，性能会提高1％至5％。但是executor内存不足时,设置的太大,就会造成OOM导致宕机 | 1.4.0         |
| `spark.reducer.maxReqsInFlight`                              | Int.MaxValue      | 参数建议：该参数用于设置fetch block时的连接数大小。如果集群节点数量很多，reducer在从其他节点fetch shuffle数据时可能会建立很多远程连接，严重时可能会导致该节点瘫痪。避免节点因为请求量过大而瘫痪掉，当并行度大到一定程度，N个[reduce](https://so.csdn.net/so/search?q=reduce&spm=1001.2101.3001.7020) task去同一个节点拉数据，</br>调优建议： | 2.0.0         |
| `spark.reducer.maxBlocksInFlightPerAddress`                  | Int.MaxValue      | 参数建议：该参数用于限制每个reduce任务从给定主机端口fetch的block数量。</br>调优建议： | 2.2.1         |
| <font color="red">`spark.shuffle.compress`</font>            | true              | 参数建议：是否压缩map的输出文件。启用后会采用spark.io.compression.codec格式进行压缩。</br>调优建议： | 0.6.0         |
| <font color="red">`spark.shuffle.file.buffer`</font>         | 32k               | 参数建议：该参数表示每个shuffle文件输出流的内存缓冲区大小（以KiB为单位）。增加此参数大小，可以降低在创建中间shuffle文件时的，磁盘寻址和系统调用的次数，提高性能</br>调优建议：不够用的话可每次翻倍，如64k。 | 1.4.0         |
| <font color="red">`spark.shuffle.io.maxRetries`</font>       | 3                 | 参数建议：（仅限Netty）`shuffle read`任务从`shuffle write`任务那里节点正在拉自己的数据，如果网络由于异常拉失败而失败，它将自动重试。 此参数表示可以重试的最大次数。 如果在指定的次数内进行或不成功，则可能导致作业失败。在遇到长时间GC或短暂网络连接问题时，这种重试逻辑可以提高shuffle的稳定性。</br>调优建议：对于那些包含耗时的`shuffle`的作业，建议增加最大重试次数（例如60次），以避免由于诸如JVM或网络的完整gc之类的因素而导致数据失败。 不稳定。 在实践中发现，对于大量数据（数十亿到数十亿的`shuffle`过程），调整参数可以大大提高稳定性。 | 1.2.0         |
| <font color="red">`spark.shuffle.io.numConnectionsPerPeer`</font> | 1                 | 参数建议：（仅限Netty）在大集群中，为了减少建立连接，主机之间的连接可以被重复使用。</br>调优建议：对于节点少但磁盘多的集群，可能导致并发度不够。可以增加此参数的值。 | 1.2.1         |
| <font color="red">`spark.shuffle.io.preferDirectBufs`</font> | true              | 参数建议：（仅限Netty）启用堆外内存，可以避免shuffle过程的频繁gc，如果堆外内存非常紧张，则可以考虑关闭这个选项</br>调优建议： | 1.2.0         |
| <font color="red">`spark.shuffle.io.retryWait`</font>        | 5s                | 参数建议：两次fetch之间需要等待多长时间。重试导致的延迟最大为：maxRetries * retryWait</br>调优建议： | 1.2.1         |
| `spark.shuffle.io.backLog`                                   | -1                | 参数建议：shuffle服务的accept队列的长度。</br>调优建议：对于比较大的应用程序可以考虑增大此参数，避免因为accept对量满了而drop掉某些连接。 | 1.1.1         |
| <font color="red">`spark.shuffle.service.enabled`</font>     | false             | 参数建议：是否启用external shuffle服务。</br>调优建议：      | 1.2.0         |
| <font color="red">`spark.shuffle.service.port`</font>        | 7337              | 参数建议：external shuffle服务运行的端口。</br>调优建议：    | 1.2.0         |
| <font color="red">`spark.shuffle.service.index.cache.size`</font> | 100m              | 参数建议：缓存Shuffle服务的索引文件的内存大小</br>调优建议： | 2.3.0         |
| `spark.shuffle.maxChunksBeingTransferred`                    | Long.MAX_VALUE    | 参数建议：shuffle服务允许同时传输的chunk的最大数量。</br>调优建议： | 2.3.0         |
| <font color="red">`spark.shuffle.sort.bypassMergeThreshold`</font> | 200               | 参数建议：当ShuffleManager为SortShuffleManager时，如果shuffle read task的数量小于这个阈值（默认是200），则shuffle write过程中不会进行排序操作，而是直接按照未经优化的HashShuffleManager的方式去写数据，但是最后会将每个task产生的所有临时磁盘文件都合并成一个文件，并会创建单独的索引文件。</br>调优建议：当你使用SortShuffleManager时，如果的确不需要排序操作，那么建议将这个参数调大一些，大于shuffle read task的数量。那么此时就会自动启用bypass机制，map-side就不会进行排序了，减少了排序的性能开销。但是这种方式下，依然会产生大量的磁盘文件，因此shuffle write性能有待提高。 | 1.1.1         |
| <font color="red">`spark.shuffle.spill.compress`</font>      | true              | 参数建议：在shuffle过程中，是否压缩溢写的文件。启用后会采用spark.io.compression.codec格式进行压缩。</br>调优建议： | 0.9.0         |
| `spark.shuffle.accurateBlockThreshold`                       | 100 * 1024 * 1024 | 参数建议：HighlyCompressedMapStatus 中记录 shuffle blcok 准确大小的阈值，当 block 小于该值则用平均值代替。</br>调优建议： | 2.2.1         |
| `spark.shuffle.registration.timeout`                         | 5000              | 参数建议：向external shuffle服务注册时的超时时间，单位ms。</br>调优建议： | 2.3.0         |
| `spark.shuffle.registration.maxAttempts`                     | 3                 | 参数建议：向external shuffle服务注册失败时，最多尝试多少次。</br>调优建议： | 2.3.0         |

##### spark ui相关配置

TODO

##### 压缩和序列化相关配置

| Property Name                                                | Default                                     | Meaning                                                      | Since Version |
| :----------------------------------------------------------- | :------------------------------------------ | :----------------------------------------------------------- | :------------ |
| <font color="red">`spark.broadcast.compress`</font>          | true                                        | 参数说明：在传递广播变量前是否压缩它。若启用，采用spark.io.compression.codec格式进行压缩。</br>调优建议：开启 | 0.6.0         |
| <font color="red">`spark.checkpoint.compress`</font>         | false                                       | 参数说明：是否压缩RDD的checkpoint。若启用，采用spark.io.compression.codec格式进行压缩。</br>调优建议：开启 | 2.2.0         |
| <font color="red">`spark.io.compression.codec`</font>        | lz4                                         | 参数说明：用来压缩内部数据（如RDD partition, event log, broadcast variable和shuffle输出）的格式。默认地，Spark提供四中压缩格式：lz4`, `lzf`, `snappy`, and `zstd。</br>调优建议： | 0.8.0         |
| <font color="red">`spark.io.compression.lz4.blockSize`</font> | 32k                                         | 参数说明：当压缩格式为LZ4时，LZ4压缩使用的block大小。</br>调优建议：更小的block size可以降低LZ4压缩时使用的shuffle memory，但会增加溢写次数。建议32k~512k。 | 1.4.0         |
| <font color="red">`spark.io.compression.snappy.blockSize`</font> | 32k                                         | 参数说明：参数说明：当压缩格式为snappy时，snappy压缩使用的block大小。</br>调优建议：更小的block size可以降低snappy压缩时使用的shuffle memory，但会增加溢写次数。建议32k~512k。</br>调优建议： | 1.4.0         |
| `spark.io.compression.zstd.level`                            | 1                                           | 参数说明：zstd压缩时的压缩级别。增大此参数可以提高压缩率，但会更消耗CPU和内存。</br>调优建议： | 2.3.0         |
| `spark.io.compression.zstd.bufferSize`                       | 32k                                         | 参数说明：zstd压缩时的buffer大小。更小的buffer size可以降低zstd压缩时使用的shuffle memory，但会增加压缩时的开销。</br>调优建议： | 2.3.0         |
| `spark.kryo.classesToRegister`                               | (none)                                      | 参数说明：如果使用Kryo序列化，可提供一个以逗号分隔的自定义类名列表，以便向Kryo注册这些类。</br>调优建议： | 1.2.0         |
| `spark.kryo.referenceTracking`                               | true                                        | 参数说明：在使用Kryo序列化数据时是否跟踪对同一对象的引用，</br>调优建议： | 0.8.0         |
| <font color="red">`spark.kryo.registrationRequired`</font>   | false                                       | 参数说明：在使用kryo序列化数据时，是否严格要求这些类必须向kryo注册。若为false，kryo仍可以序列化这些未注册的类，但是性能开销会很大。</br>调优建议：设置为true，然后向kryo注册需要序列化的类，可以提高序列化的性能（更短的时间，更小的字节数） | 1.1.0         |
| `spark.kryo.registrator`                                     | (none)                                      | 参数说明：配置此参数的前提是，要以自定义的方式向kryo注册一些类（默认的注册方式是spark.kryo.classesToRegister）。此参数值必须是[`KryoRegistrator`](https://spark.apache.org/docs/3.0.2/api/scala/org/apache/spark/serializer/KryoRegistrator.html)的子类</br>调优建议： | 0.5.0         |
| `spark.kryo.unsafe`                                          | false                                       | 参数说明：是否使用基于不安全的Kryo序列化器。使用基于不安全的IO可以大大加快速度。</br>调优建议： | 2.1.0         |
| <font color="red">`spark.kryoserializer.buffer.max`</font>   | 64m                                         | 参数说明：kryo序列化的最大buffer大小。范围：需要序列化的任何对象 < buffer.max < 2048m。</br>调优建议：如果出现"buffer limit exceeded"的报错，需要增大此值。 | 1.4.0         |
| `spark.kryoserializer.buffer`                                | 64k                                         | 参数说明：kryo序列化的初始buffer大小。***注意：一个worker节点的一个core会有一个buffer。***最大可增大到spark.kryoserializer.buffer.max</br>调优建议： | 1.4.0         |
| `spark.rdd.compress`                                         | false                                       | 参数说明：是否压缩序列化的RDD partition。可以节省空间，但会消耗更多的CPU资源。启用后，使用spark.io.compression.codec格式进行压缩。</br>调优建议： | 0.6.0         |
| <font color="red">`spark.serializer`</font>                  | org.apache.spark.serializer. JavaSerializer | 参数说明：用来序列化对象的类。序列化之后的可以通过网络传输到其他，也可以缓存。可以是任何org.apache.spark.Serializer的子类。</br>调优建议：默认的Java序列化器很慢。推荐使用org.apache.spark.serializer.KryoSerializer | 0.5.0         |
| `spark.serializer.objectStreamReset`                         | 100                                         | 参数说明：当使用org.apache.spark.serializer.JavaSerializer进行序列化时，序列化器会缓存对象以防止写入冗余数据，但这会停止这些对象的垃圾收集。通过调用“reset”，可以从序列化器flush数据，并允许收集旧对象。要关闭此定期重置，请将其设置为-1。默认情况下，它将每100个对象reset一次序列化程序。</br>调优建议： | 1.0.0         |

##### 内存管理相关配置

| Property Name                                               | Default | Meaning                                                      | Since Version |
| :---------------------------------------------------------- | :------ | :----------------------------------------------------------- | :------------ |
| <font color="red">**`spark.memory.fraction`**</font>        | 0.6     | 参数说明：用于execution和storage的内存，默认情况下，内存大小为(heap space - 300MB)*spark.memory.fraction。300MB是预留的内存。值越低，溢写和驱逐数据（从内存中）的频率会更高。</br>调优建议： | 1.6.0         |
| <font color="red">**`spark.memory.storageFraction`**</font> | 0.5     | 参数说明：`spark.memory.fraction`中用于storage的memory比例。此内存中的数据不会被驱逐。</br>调优建议：此值越大，则用于execution的内存越少，execution相关的数据可能会更频繁的溢写到磁盘。 | 1.6.0         |
| <font color="red">**`spark.memory.offHeap.enabled`**</font> | false   | 参数说明：如果为true，Spark将尝试将堆外内存用于某些操作。如果启用了堆外内存使用，则spark.memory.offHeap.size必须为正。</br>调优建议： | 1.6.0         |
| <font color="red">**`spark.memory.offHeap.size`**</font>    | 0       | 参数说明：堆外内存的大小</br>调优建议：                      | 1.6.0         |
| `spark.storage.replication.proactive`                       | false   | 参数说明：是否启用RDD 块复制。</br>调优建议：一般不用改      | 2.2.0         |
| `spark.cleaner.periodicGC.interval`                         | 30min   | 参数说明：控制GC触发的频率。只有垃圾回收弱引用时，context清理器会触发。</br>调优建议：一般不用改 | 1.6.0         |
| `spark.cleaner.referenceTracking`                           | true    | 参数说明：是否启用context清理。</br>调优建议：一般不用改     | 1.0.0         |
| `spark.cleaner.referenceTracking.blocking`                  | true    | 参数说明：控制清理器的线程是否会阻塞cleanup task。</br>调优建议：一般不用改 | 1.0.0         |
| `spark.cleaner.referenceTracking.blocking.shuffle`          | false   | 参数说明：控制清理器线程是否会阻塞shuffle cleanup task任务</br>调优建议：一般不用改 | 1.1.1         |
| `spark.cleaner.referenceTracking.cleanCheckpoints`          | false   | 参数说明：引用如果超出范围，是否清除checkpoint文件。</br>调优建议：一般不用改 | 1.4.0         |

##### execution相关配置

| Property Name                                                | Default                                                      | Meaning                                                      | Since Version |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :------------ |
| `spark.broadcast.blockSize`                                  | 4m                                                           | 参数说明：TorrentBroadcastFactory的块大小。值太大会降低广播变量时的并行度；值太小会影响BlockManager的性能。</br>调优建议：一般不修改。 | 0.5.0         |
| `spark.broadcast.checksum`                                   | true                                                         | 参数说明：在广播过程中是否计算校验和。用来检测广播过程中哪些块损坏了。代价是计算和发送更多的数据</br>调优建议：一般不修改。 | 2.1.1         |
| <font color="red">`spark.executor.cores`</font>              | 1 in YARN mode,  standalone和Mesos coarse-grained modes   是worker节点的所有core | 参数说明：executor可用的core数</br>调优建议：任务级别调整。  | 1.0.0         |
| <font color="red">`spark.default.parallelism`</font>         | 对于reduceByKey和join等分布式shuffle操作，默认值为父RDD中的最大分区数。</br>local模式：本地机器的core数；</br>Mesos模式：8；</br>其他模式：所有executor上的总核数与2的最大值。如配置Executor的数量为5，每个Executor的CPU Core数量为2，executor上的总核数10，则默认并行度为`Max(10,2)=10`。 | 参数说明：通过join、reduceByKey和parallelize等transformation算子返回的RDD中的默认分区数。</br>调优建议： | 0.5.0         |
| `spark.executor.heartbeatInterval`                           | 10s                                                          | 参数说明：executor向driver发送心跳的间隔时间。通过心跳，driver可以知道executor是否存活，且每次心跳会更新executor的一些指标数据。</br>调优建议：spark.executor.heartbeatInterval应显著小于spark.network.timeout | 1.1.0         |
| `spark.files.fetchTimeout`                                   | 60s                                                          | 参数说明：从driver获取 通过`SparkContext.addFile()`添加的文件时使用的超时时间。</br>调优建议：一般不修改。 | 1.0.0         |
| `spark.files.useFetchCache`                                  | true                                                         | 参数说明：fetch文件时是否通过本地缓存进行共享，同一个application的executor如果在同一个worker节点，那么这些executor是可以共享本地缓存中的文件，效率更高。</br>调优建议：一般不修改。 | 1.2.2         |
| `spark.files.overwrite`                                      | false                                                        | 参数说明：当目标文件存在且其内容与源文件不匹配时，是否覆盖通过`SparkContext.addFile()`添加的文件。</br>调优建议：一般不修改。 | 1.0.0         |
| `spark.files.maxPartitionBytes`                              | 134217728 (128 MiB)                                          | 参数说明：在读取文件时，会将文件分割为partition，一个partition的最大字节数为此值。</br>调优建议：值太大会降低并发度。一般不修改。 | 2.1.0         |
| `spark.files.openCostInBytes`                                | 4194304 (4 MiB)                                              | 参数说明：打开文件的成本。表示同一时间可扫描的字节数。</br>调优建议：一般不修改。 | 2.1.0         |
| `spark.hadoop.cloneConf`                                     | false                                                        | 参数说明：如果设置为true，则为每个task克隆一个新的Hadoop配置对象。应启用此选项以解决配置线程安全问题。开启会降低性能</br>调优建议：一般不修改。 | 1.0.3         |
| `spark.hadoop.validateOutputSpecs`                           | true                                                         | 参数说明：</br>调优建议：一般不修改。                        | 1.0.1         |
| `spark.storage.memoryMapThreshold`                           | 2m                                                           | 参数说明：从磁盘读取数据块时，Spark内存映射的数据块大小。</br>调优建议：一般不修改。 | 0.9.2         |
| `spark.hadoop.mapreduce.fileoutputcommitter.algorithm.version` | 1                                                            | 参数说明：</br>调优建议：一般不修改。                        |               |

##### executor指标相关配置



##### 网络相关配置

| Property Name                                        | Default                            | Meaning                                                      | Since Version |
| :--------------------------------------------------- | :--------------------------------- | :----------------------------------------------------------- | :------------ |
| <font color="red">`spark.rpc.message.maxSize`</font> | 128                                | 参数说明：executor和driver间RPC通信时传递的数据量大小。如果任务数过多（上千）</br>调优建议：如果报错提示此值过小，可以增大此值。如256，512等。 | 2.0.0         |
| `spark.blockManager.port`                            | (random)                           | 参数说明：</br>调优建议：一般不修改。                        | 1.1.0         |
| `spark.driver.blockManager.port`                     | (value of spark.blockManager.port) | 参数说明：</br>调优建议：一般不修改。                        | 2.1.0         |
| `spark.driver.bindAddress`                           | (value of spark.driver.host)       | 参数说明：</br>调优建议：一般不修改。                        | 2.1.0         |
| `spark.driver.host`                                  | (local hostname)                   | 参数说明：spark standalone模式。</br>调优建议：一般不修改。  | 0.7.0         |
| `spark.driver.port`                                  | (random)                           | 参数说明：spark standalone模式。</br>调优建议：一般不修改。  | 0.7.0         |
| `spark.rpc.io.backLog`                               | 64                                 | 参数说明：</br>调优建议：一般不修改。                        | 3.0.0         |
| <font color="red">`spark.network.timeout`</font>     | 120s                               | 参数说明：所有网络交互的默认超时时间。如果配置了`spark.storage.blockManagerSlaveTimeoutMs`</br>, `spark.shuffle.io.connectionTimeout`</br>, `spark.rpc.askTimeout` or `spark.rpc.lookupTimeout`</br>则对应的功能的超时时间不为spark.network.timeout。</br>调优建议：一般不修改。 | 1.3.0         |
| `spark.network.io.preferDirectBufs`                  | true                               | 参数说明：</br>调优建议：一般不修改。                        | 3.0.0         |
| `spark.port.maxRetries`                              | 16                                 | 参数说明：</br>调优建议：一般不修改。                        | 1.1.1         |
| `spark.rpc.numRetries`                               | 3                                  | 参数说明：</br>调优建议：一般不修改。                        | 1.4.0         |
| `spark.rpc.retry.wait`                               | 3s                                 | 参数说明：</br>调优建议：一般不修改。                        | 1.4.0         |
| `spark.rpc.askTimeout`                               | `spark.network.timeout`            | 参数说明：</br>调优建议：一般不修改。                        | 1.4.0         |
| `spark.rpc.lookupTimeout`                            | 120s                               | 参数说明：</br>调优建议：一般不修改。                        | 1.4.0         |
| `spark.network.maxRemoteBlockSizeFetchToMem`         | 200m                               | 参数说明：</br>调优建议：一般不修改。                        | 3.0.0         |

##### 调度相关配置

TODO

##### barrier execution mode

TODO

##### 动态资源调度配置

| Property Name                                              | Default                                | Meaning                                                      | Since Version |
| :--------------------------------------------------------- | :------------------------------------- | :----------------------------------------------------------- | :------------ |
| `spark.dynamicAllocation.enabled`                          | false                                  | 参数说明：是否启用动态资源调度，会根据工作负载动态调整此application的executor的数量</br>调优建议：可以开启。 | 1.2.0         |
| `spark.dynamicAllocation.executorIdleTimeout`              | 60s                                    | 参数说明：当spark.dynamicAllocation.enabled=true后，若executor的空闲时间超过此值，则释放此executor。</br>调优建议：一般不修改。 | 1.2.0         |
| `spark.dynamicAllocation.cachedExecutorIdleTimeout`        | infinity                               | 参数说明：当spark.dynamicAllocation.enabled=true后，若缓存了数据的executor的空闲时间超过此值，则释放此executor。</br>调优建议：一般不修改。240s | 1.4.0         |
| `spark.dynamicAllocation.initialExecutors`                 | `spark.dynamicAllocation.minExecutors` | 参数说明：当spark.dynamicAllocation.enabled=true后，此值表示应用程序初始的executor数量。若通过`--num-executors`指定了executor的数量，则初始的exectuor数量为max(此项配置的值, `--num-executor`的值)</br>调优建议：一般不修改。 | 1.3.0         |
| `spark.dynamicAllocation.maxExecutors`                     | infinity                               | 参数说明：当spark.dynamicAllocation.enabled=true后，executor的数量上限</br>调优建议：根据具体修改，如10 | 1.2.0         |
| `spark.dynamicAllocation.minExecutors`                     | 0                                      | 参数说明：当spark.dynamicAllocation.enabled=true后，executor的数量下限</br>调优建议：根据具体修改，1 | 1.2.0         |
| `spark.dynamicAllocation.executorAllocationRatio`          | 1                                      | 参数说明：默认情况下，动态资源调度会根据task量来申请足够多的executor保证最大化并行度。但是这种情况可能存在资源浪费（因为某些executor可能不会做任何事情）。通过这个比率来控制申请的executor的数量。预计申请的executor数*此值。一般为1.0或0.5。**当此参数计算出来的executor数量与spark.dynamicAllocation.maxExecutors或spark.dynamicAllocation.minExecutors的值冲突时，以后两者为准。**</br>调优建议：一般不修改。 | 2.4.0         |
| `spark.dynamicAllocation.schedulerBacklogTimeout`          | 1s                                     | 参数说明：当spark.dynamicAllocation.enabled=true后，如果pending的task的积压时间超过此值，则为其申请新的executor。</br>调优建议：一般不修改。 | 1.2.0         |
| `spark.dynamicAllocation.sustainedSchedulerBacklogTimeout` | `schedulerBacklogTimeout`              | 参数说明：同schedulerBacklogTimeout，是申请了新executor之后继续申请的间隔，默认=schedulerBacklogTimeout（第二次及之后）</br>调优建议：一般不修改。 | 1.2.0         |
| `spark.dynamicAllocation.shuffleTracking.enabled`          | `false`                                | 参数说明：（实验性质）是否启用shuffle文件跟踪，它允许动态资源调度可以不依赖external shuffle服务。此配置不会回收保存了shuffle数据的executor。</br>调优建议：一般不修改。 | 3.0.0         |
| `spark.dynamicAllocation.shuffleTracking.timeout`          | `infinity`                             | 参数说明：当spark.dynamicAllocation.shuffleTracking.enabled=true后，控制保存shuffle数据的executor超时时间，默认使用GC垃圾回收控制释放。如果有时候GC不及时，配置此参数后，即使executor上存在shuffle数据，也会被回收。</br>调优建议：一般不修改。 | 3.0.0         |

##### 线程配置

TODO

##### Spark SQL相关配置

TODO

##### Spark Streaming相关配置

TODO

##### SparkR相关配置

TODO

##### GraphX相关配置

TODO

##### 部署相关配置

TODO

##### 集群管理器相关配置

TODO

### 环境变量

Spark的环境变量定义在`$SPARK_HOME/conf/spark-env.sh`脚本中（windows系统则在`$SPARK_HOME/conf/spark-env.cmd`）。

默认情下，当Spark安装好之后，`$SPARK_HOME/conf/spark-env.sh`文件并不存在，可以通过复制`$SPARK_HOME/conf/spark-env.sh.template`文件来创建`$SPARK_HOME/conf/spark-env.sh`。需要确保`$SPARK_HOME/conf/spark-env.sh`文件是可执行的。

| Environment Variable    | Meaning                                                      |
| :---------------------- | :----------------------------------------------------------- |
| `JAVA_HOME`             | Location where Java is installed (if it's not on your default `PATH`). |
| `PYSPARK_PYTHON`        | Python binary executable to use for PySpark in both driver and workers (default is `python2.7` if available, otherwise `python`). Property `spark.pyspark.python` take precedence if it is set |
| `PYSPARK_DRIVER_PYTHON` | Python binary executable to use for PySpark in driver only (default is `PYSPARK_PYTHON`). Property `spark.pyspark.driver.python` take precedence if it is set |
| `SPARKR_DRIVER_R`       | R binary executable to use for SparkR shell (default is `R`). Property `spark.r.shell.command` take precedence if it is set |
| `SPARK_LOCAL_IP`        | IP address of the machine to bind to.                        |
| `SPARK_PUBLIC_DNS`      | Hostname your Spark program will advertise to other machines. |



<font color="red">***注意：当以Spark on YARN的集群模式运行时，环境变量需要在`$SPARK_HOME/conf/spark-defaults.xml`文件中通过`spark.yarn.appMasterEnv.[EnvironmentVariableName]`的形式来设置。这是因为在`spark-env.sh`中设置的环境变量并不能在YARN的Application Master（AM）进程中生效。***</font>

### 配置日志格式

Spark使用log4j来记录日志，通过`$SPARK_HOME/conf/log4j.properties`配置文件可以决定Spark如何记录日志。

### 覆盖配置目录

修改`SPARK_CONF_DIR`环境变量的值，就可以让Spark读取该目录下的配置文件（`spark-defaults.conf`, `spark-env.sh`, `log4j.properties`等），而不从默认的`SPARK_HOME/conf`目录下读取配置。

### 继承Hadoop集群的配置

要让Spark能够从HDFS中读取和写入数据，那么要保证在Spark的classpath中存在如下两个配置文件：

- core-site.xml
- hdfs-site.xml

要使这些文件对Spark可见，请将`$spark_HOME/CONF/Spark-env.sh`中的`HADOOP_CONF_DIR`设置为`$HADOOP_HOME/etc/hadoop/conf`。

### 自定义Hadoop/Hive的配置

不同的application可能需要不同的Hadoop/Hive的client端配置。在Spark on YARN模式下，这个时候不能通过修改Spark的classpath中的`hdfs-site.xml`, `core-site.xml`, `yarn-site.xml`, `hive-site.xml`文件来适配所有application。修改了这些配置，也可能会影响cluster端的配置参数。

更好的做法是:

- 使用`spark.hadoop.*`这种spark hadoop属性的方式在应用程序中指定需要设置的参数

  下面的代码表示设置了一个hadoop属性`abc.def=xyz`

  ```scala
  val conf = new SparkConf().set("spark.hadoop.abc.def", "xyz")
  ```

  

- 使用`spark.hive.*`

  下面的代码表示设置了一个hive属性`abc=xyz`

  ```scala
  val conf = new SparkConf().set("spark.hadoop.abc", "xyz")
  ```

这类`spark.hadoop.*`和`spark.hive.*`属性也可以在`$spark_HOME/conf/spark-defaults.conf`中设置。

还可以在`spark-submit`的命令行显式地指定这里参数：

```shell
./bin/spark-submit \ 
  --name "My app" \ 
  --master local[4] \  
  --conf spark.eventLog.enabled=false \ 
  --conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps" \ 
  --conf spark.hadoop.abc.def=xyz \
  --conf spark.hive.abc=xyz
  myApp.jar
```



### 自定义资源调度配置

TODO

## 1.12 Spark调优

参考：https://spark.apache.org/docs/3.0.2/tuning.html

Spark程序受到：CPU、网络带宽、内存的资源限制。

- 数据序列化

  选择合理的序列化格式，可以减少内存的使用，也可以减少网络传输中的数据量大小，提高应用程序执行效率。

  ```scala
  val conf = new SparkConf()
               .setMaster("local[2]")
               .setAppName("CountingSheep")
  val sc = new SparkContext(conf)
  ```

  

- 内存调优

### 数据序列化

分布式应用中，序列化扮演这很重要的角色。如果序列化很慢，或者序列化之后占用大量的字节，这会严重降低计算的效率。

Spark提供了两个序列化的库：

- Java serialization

  默认情况下，Spark使用Java的`ObjectOutputStream`框架对对象进行序列化，兼容任何实现`Java.io.Serializable`的类。还可以通过扩展`java.io.Externalizable`来控制序列化的性能。Java序列化是灵活的，但通常非常慢。

- Kryo serialization

  Spark还可以使用Kryo库（版本4）更快地序列化对象。Kryo比Java序列化快得多，也更紧凑（通常高达10倍），但它不支持所有的Serializable类型，需要您预先注册程序中使用的类以获得最佳性能。



切换序列化类为Kryo。

```scala
val conf = new SparkConf().setMaster(...).setAppName(...)
conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
```

这样，在worker节点shuffle序列化时和RDD序列化到磁盘时，都使用Kryo来序列化。

之所以不把Kryo设置为默认的序列化器，是因为需要在手动注册需要kryo来序列化 的类。如下所示：

```scala
val conf = new SparkConf().setMaster(...).setAppName(...)
conf.registerKryoClasses(Array(classOf[MyClass1], classOf[MyClass2]))
val sc = new SparkContext(conf)
```



### 内存调优

在内存调优中，主要考虑三个因素：

- 对象使用的内存量
- 访问这些对象的成本
- 垃圾回收的开销

默认情况下，Java对象访问速度很快，但很容易比字段中的“原始”数据多消耗2-5倍的空间。这有几个原因：

- 每个不同的Java对象都有一个“对象头”，大约16个字节，包含指向其类的指针等信息。对于一个数据很少的对象（比如一个Int字段），这可能比数据大。
- Java字符串在原始字符串数据上有大约40个字节的开销（因为它们将其存储在一个Chars数组中，并保留额外的数据，例如长度），并且由于字符串内部使用UTF-16编码，因此将每个字符存储为两个字节。因此，10个字符的字符串可以很容易地消耗60个字节。
- 常见的集合类，如HashMap和LinkedList，使用链接数据结构，其中每个条目都有一个“包装器”对象（例如Map.entry）。该对象不仅有一个标头，而且还有指向列表中下一个对象的指针（通常每个8个字节）。
- 基本类型的集合通常将它们存储为“装箱”对象，对应的包装器对象，例如java.lang.Integer。

#### Spark的内存管理

Spark中的内存使用可以分为两类：执行内存（execution memory）和存储内存（storage memory）。

- execution memory

  指的是在shuffle、join、sort、aggregation等计算过程中使用的内存。

- storage memory

  指的是在集群中，缓存和传递内部数据所用的内存。

在Spark中，执行和存储共享一个统一的区域（M）。当不使用执行内存时，存储可以获取所有可用内存，反之亦然。R是应用程序中storage memory的最小值，当storage memory使用的内存小于R时，会减少execution memory使用的内存，以增加storage memory使用的内存。

有几个好处：

1、不使用缓存的应用程序可以使用整个execution memory，从而避免不必要的磁盘溢写。

2、使用缓存的应用程序可以保留最小的storage memory（R），使其数据块不会被逐出。

3、这种方法为各种工作负载提供了合理的开箱即用性能，而不需用户具有内存分配的专门知识。

- `spark.memory.fraction默认值0.6。`表示M占JVM内存的比例，即execution memory和storage memory的总大小。剩下的0.4的内存用于用户的数据结构、Spark内部元数据等。
- `spark.memory.storageFraction`默认0.5。表示R占M的比例。表示用于存储的内存至少为0.5M。

#### 如何确定对象的内存使用情况

确定DataSet所需内存消耗量的最佳方法是创建RDD，将其放入缓存，然后查看web UI中的“storage”页面。该页面将告诉您RDD占用了多少内存。
要估计特定对象的内存消耗，请使用SizeEstimator的estimate 方法。这对于尝试不同的数据布局以减少内存使用，以及确定广播变量将在每个executor堆上占用的空间量非常有用。

#### 优化数据结构

减少内存消耗的方法：

避免使用增加内存消耗的JAVA特性，如基于指针的数据结构和包装对象。

- 使用合适的数据结构和基础类型。如fastutil提供了比较方便的结合类，而且兼容Java标准库。标准Java或标准Scala集合类可能消耗内存更大。
- 尽可能避免包含大量小对象和指针的嵌套结构
- key使用数字类型的id，或者枚举类，而不是String类型
- 如果内存小于32GB，可以通过设置JVM参数`-XX:+UseCompressedOops`来减少指针所占的字节（8字节减少为4字节）。可以在spark-env.sh中添加此选项

#### 序列化RDD存储

对象太大而无法进行有效存储，可以将RDD序列化之后存储，这样可以减少存储时占用的内存。

使用这种方式，Spark会将RDD的每个partition存储为一个大的字节数组。以序列化的形式存储的有点是占用的内存空间更小，缺点是访问时需要先反序列化，因此访问较慢。建议使用Kryo序列化。

#### 如何调整Spark的缓存大小

TODO

#### 如何调整Java垃圾收集器

如果GC过于频繁，可以尝试使用序列化RDD之后再缓存。

GC调优的第一步是收集垃圾收集频率和GC花费时间的统计信息。这可以通过向Java选项添加`-verbose:gc -XX:+PrintGCDetails -XX:+PPrintGCTimeStamps`来完成。请注意，这些日志将位于集群的工作节点（位于其工作目录中的stdout文件中），而不是驱动程序中。

为了进一步优化垃圾收集，我们首先需要了解JVM内存管理的一些基本信息：

- Java堆空间分为两个区域：Young和Old。年轻代保存短期的对象，而老年代保存长期的对象。
- 年轻代又划分为3个区域：Eden、Survivor1、Survivor2。
- GC流程：
  - 当Eden满了，在Eden中运行一次minor GC，Eden和Survivor1中存活的对象copy到Survivor2中
  - Survivor2变为Survivor1，Survivor1变为Survivor2
  - 当对象足够老，或者Survivor2满了，将其中的对象移动到Old
  - 当Old满了，执行Full GC。

Spark中GC调优的目标是确保Old中只存储长期的RDD，短期存活的对象存储在Young中。

- 通过收集GC统计信息来检查GC是否过多。如果在任务完成之前多次调用Full GC，则意味着没有足够的内存用于执行任务。

- 如果minor GC次数很多，而major GC次数很少，则可以增加Eden的内存大小。

- 在打印的GC统计信息中，如果OldGen快要满了，可以降低`spark.memory.fraction`比例来降低用于spark中execution memory和storage memory的内存量。另外，还可以降低YoungGen的内存比例。

- 使用`G1GC` 垃圾回收器。`-XX:+UserG1GC`。可以提高性能。如果executor的堆内存比较大，可以使用`-XX:G1HeapRegionSize`增加G1区域的内存大小。

- 举个例子，如果Task是从HDFS读取数据，那么可以按如下来估算Eden的内存大小。

  一个executor运行4个task，

  一个task读取一个block的数据

  一个block解压之后是原来的3倍大小

  HDFS的block sieze 为128MB

  那么可以设置Eden大小为：4 * 3 * 128MB。

- 监视GC的频率和时间如何随新设置而变化。

GC调优的效果要看具体的应用程序和具体分配的内存量，也是任务级别的调优了。

### 其他调优

#### 并行度

在编写Spark应用程序是，可以指定应用程序的并发度，如`SparkContext.textFile()`的第二个参数，也可以设置`spark.default.paralelism`来修改spark应用程序的默认并发度。建议是每个CPU核心处理2-3个Task。

#### 列出Input Path的并行度

当作业输入包含大量目录时，您可能还需要增加目录列表的并行性，否则该过程可能会花费很长时间，尤其是在针对S3这样的对象存储时。如果您的工作在RDD上使用Hadoop输入格式（例如，通过`SparkContext.sequenceFile`），则并行度通过`spark.Hadoop.mapreduce.input.fileinputformat.list-status.num-threads`（当前默认值为1）控制。

对于具有基于文件的数据源的Spark SQL，可以调整`Spark.SQL.sources.parallelPartitionDiscovery.threshold`和`Spark.SQL.sesources.parallel PartitionDiscovery.parallelism`以提高列表并行性。

参考：Spark SQL调优 https://spark.apache.org/docs/3.0.2/sql-performance-tuning.html

#### reduce task的内存使用

有时候会出现某个task OutOfMemoryError的情况，比如`groupByKey`的reduce task涉及的数据量太大了。Spark的Shuffle操作（`sortByKey`, `groupByKey`,`reduceByKey`, `join`等）会在每个reduce task中构建一个`Hashtable`来进行分组。

解决方法：

提高并行度，使得每个reduce task的输入数据量变小。

#### 广播大变量

如果task需要使用driver程序中的大的对象，那么可以将此对象转换为广播变量。（通常大于20KB的对象，可以考虑转换为广播变量）。

普通变量会copy给每个task一个该变量的副本

广播变量会copy给每个executor一个该变量的副本

将普通变量转换为广播变量可以减小传输的数据量。

#### 数据位置（Data Locality）

计算和数据在同一位置，那么计算会比较快。如果计算和数据是分离的，那么需要将计算（应用程序代码）移动到数据节点，再进行计算（应为应用程序代码比数据大小，小的多）。

### 总结

Spark应用程序调优的主要问题——数据序列化和内存调优。对于大多数程序，切换到Kryo序列化，并将序列化的数据持久化将解决大多数常见的性能问题。

**数据位置是指数据与处理它的代码之间的距离**。基于数据的当前位置，有几个级别的位置。按照从最近到最远的顺序：

- `PROCESS_LOCAL`

  数据与运行代码位于同一JVM中。这是最佳位置

- `NODE_LOCAL`

  数据位于同一节点上。示例可能在同一节点上的HDFS中，或者在同一个节点上的另一个executor中。这比`PROCESS_LOCAL`稍慢，因为数据必须在进程之间传输

- `NO_PREF`

  数据可以从任何地方快速访问，并且没有位置偏好

- `RACK_LOCAL`

  数据位于同一服务器机架上。数据位于同一机架上的不同服务器上，因此需要通过网络发送，通常通过单个交换机传输

- `any` 

  数据都在网络上的其他位置，而不在同一机架中

数据和计算可以在同一节点，但是该节点计算资源不够了，此时有两种选择：

a）等待忙碌的CPU释放出来，在同一台服务器上启动计算任务

b）在其它节点启动计算任务，并移动需要的数据。

Spark通常做的是等待一段时间，希望繁忙的CPU能够释放出来。一旦超时，它就开始将数据从远处移动到空闲CPU所在节点。

## 1.13 Spark监控

参考：https://spark.apache.org/docs/3.0.2/monitoring.html

TODO

## 1.14 作业调度

参考：https://spark.apache.org/docs/3.0.2/job-scheduling.html

### 跨应用程序的调度

#### 静态资源调度

- 独立模式

  默认情况下，提交到`Spark Standalone Cluster`的应用程序将以FIFO顺序运行，每个应用程序将尝试使用所有可用节点。可以通过在应用程序中设置`spark.cores.max`配置属性来限制应用程序使用的节点数，`spark.deploy.defaultCores`设置的应用程序在默认情况下使用的节点数。每个应用程序的·spark.executor.memory·设置还控制executor的内存使用。

- Mesos

  要在Mesos上使用静态分区，请将spark.Mesos.coarse配置属性设置为true，并可以选择将spark.cores.max设置为限制每个应用程序在独立模式下的资源共享。您还应该设置spark.executor.memory来控制executor内存。

- YARN

  `--num-executors` 表示Spark YARN 客户端控制分配多少个executor来执行应用程序，对应的Spark 配置属性为`spark.executor.instances`

  `--executor-memory` 表示分配给每个executor的内存大小。对应的Spark 配置属性为`spark.executor.memory`

  `--executor-cores` 表示分配给每个executor的CPU核心数。对应的Spark 配置属性为`spark.executor.cores`

***请注意，目前没有一种模式提供跨应用程序的内存共享。如果您希望以这种方式共享数据，我们建议运行一个服务器应用程序，该应用程序可以通过查询相同的RDD来服务多个请求。***

#### 动态资源调度

Spark提供了一种基于工作负载动态调整应用程序占用的资源的机制。这意味着，如果不再使用资源，您的应用程序可能会将其返还给集群，并在稍后有需求时再次请求资源。如果多个应用程序共享Spark集群中的资源，此功能特别有用。

默认情况下，此功能是禁用的，在`Spark Standalone 模式`，`YARN 模式`、`Mesos粗粒度模式`、`K8s模式`上都可用。

##### 配置动态资源调度

- 方法一

  `spark.dynamicAllocation.enabled` 为 true

  `spark.dynamicAllocation.shuffleTracking.enabled` 为 true

- 方法二

  `spark.dynamicAllocation.enabled` 为 true

  `spark.shuffle.service.enabled` 为 true

  shuffle tracking 或 external shuffle service的目的是：允许删除executor，但不删除shuffle write的文件。这样既释放了计算资源，shuffle read的 task仍能够获取到要读取的shuffle文件。

  启动`external shufffle service`的方式：

  - `Spark Standalone模式`：将`spark.shuffle.service.enabled` 设为 true即可。

  - `Mesos coarse-grained模式`：在``spark.shuffle.service.enabled`设置为true的所有worker节点上运行`$SPARK_HOME/sbin/start-Mesos-shuffle-service.sh`

  - `YARN模式`：https://spark.apache.org/docs/3.0.2/running-on-yarn.html#configuring-the-external-shuffle-service

    https://cloud.tencent.com/document/product/589/38242

    （1）找到`$SPARK_HOME/common/network-yarn/target/scala-<version>`或`$SPARK_HOME/yarn`下的`spark-<version>-yarn-shuffle.jar`

    （2）将此`spark-<version>-yarn-shuffle.jar`添加到所有NodeManager的classpath中。将 `spark-<version>-yarn-shuffle.jar` 拷贝到 `/usr/local/service/hadoop/share/hadoop/yarn/lib` 下

    （3）在`yarn-site.xml`中，在`yarn.nodemanager.aux-services`中增加`spark_shuffle`。`yarn.nodemanager.aux-services.spark_shuffle.class` 增加 `org.apache.spark.network.yarn.YarnShuffleService`

    （4）`etc/hadoop/yarn-env.sh`中设置`YARN_HEADSIZE`来增加NodeManager的堆内存，避免shuffle期间出现GC问题。

    （5）`spark.yarn.shuffle.stopOnFailure` 设置为 false。

    （6）重启集群内所有NodeManager。

    （7）上面是启动了外部shuffle服务，还可以通过配置`spark.dynamicAllocation.*`  和 `spark.shuffle.service.*`的配置项，来进一步优化。

##### 资源分配策略

需要一些规则来告诉Spark，什么时候移除executor，什么时候申请executor。

##### 请求executor的策略

开启了`spark动态资源调度`之后，如果Spark应用程序存在`pending`的task时，会请求executor。

资源申请时一轮一轮的。当task处于`pending`状态超过`spark.dynamicAllocation.schedulerBacklogTimeout`秒，就会触发申请executor。只要队列中存在`pending`的task，则每隔`spark.dynamicAllocation.schedulerBacklogTimeout`秒就会申请executor。此外，每一轮请求的executor数量比上一轮呈指数增长。例如，一个应用程序将在第一轮中添加1个executor，然后在随后的轮中添加2个、4个、8个等executor。

##### 删除executor的策略

一个Spark应用程序当它的某个executor的空闲时间超过`spark.dynamicAllocation.executorIdleTimeout`秒之后，就会移除这个executor。

##### 优雅地退役executor

在开启`Spark动态资源调度`之前，只有当executor所属的因公程序退出了，或者改executor执行失败了，才会退出。不管是哪种情况，这个executor删掉了都影响。

在开启`Spark动态资源调度`之后，如果删除了某executor，然后应用程序需要访问这个executor的计算结果，则需要重新计算。这是不合适的，因此Spark需要一种机制，通过保留executor的状态来确认是否应该优雅地删除executor。

上面这个问题在Spark shuffle过程尤为重要。在某executor的map任务中shuffle write将map的输出写入到本地磁盘。然后，当其他executor尝试获取这些shuffle文件时，此executor会充当一个文件服务器的角色。

保存shuffle文件的解决方案：

在Spark 1.2引入了`external shuffle service`。此服务是一个长期运行的进程，它运行在集群的每个worker节点上。此进程独立于Spark应用程序和executor。当`external shuffle service`服务启用后，Spark的executor就可以从`external shuffle service`服务获取shuffle 文件，而不是从其他executor去获取。

默认情况下，包含缓存数据的executor是不能被删除的。因为删除了，缓存的数据就丢失了。通过配置`spark.dynamicAllocation.cachedExecutorIdleTimeout`参数，可以决定超过多少秒之后可以删除包含缓存数据的executor。

### 应用程序内部的调度

在一个Spark 应用程序中，会存在多个jobs。

默认情况下，Spark以FIFO的方式调度应用程序中的jobs。第一个job先获得资源，如果还有剩余资源，且job2不依赖与job1，那么job2也可以立马获得资源。优点：策略简单；缺点：若大任务多，会阻塞后续的小任务，小任务会被饿死。

从Spark 0.8开始，应用程序内的jobs支持以fair share的方式调度。Spark以"round robin"（轮询）的方式分配多个jobs中的task，这样所有的jobs都能获得大致相同的集群资源。多租户场景建议使用此种调度方式。优点：小任务不会饿死，调度效率高；缺点：调度逻辑较复杂。

#### 如何启用应用程序内的job的公平调度呢？

```scala
val conf = new SparkConf().setMaster(...).setAppName(...)
conf.set("spark.scheduler.mode", "FAIR")
val sc = new SparkContext(conf)
```

此参数可以配置任务级别，也可以集群级别的。

#### 公平调度池（Fair Scheduler Pools）

公平调度程序还支持将作业分组到池中，并为每个池设置不同的调度选项（例如权重）。这对于为更重要的作业创建“高优先级”池非常有用，例如，或者将每个用户的作业分组在一起，无论用户有多少并发作业，都给用户相等的份额，而不是给作业相等的份额。这种方法是以Hadoop Fair Scheduler为模型的。

设置`spark.scheduler.pool`属性后，在此线程中提交的所有作业（通过在该线程中调用RDD.save、count、collect等）都将使用此池名称。**该设置是针对每个线程的，以便于让一个线程代表同一用户运行多个作业**。如果要清除线程关联的池，只需调用：

```scala
// Assuming sc is your SparkContext variable
sc.setLocalProperty("spark.scheduler.pool", "pool1")
```

l清除线程关联的池：

```scala
sc.setLocalProperty("spark.scheduler.pool", null)
```

#### 默认pool

默认情况下，每个池获得的集群份额相等（也与默认池中的每个作业的份额相等），但在每个池中，作业按FIFO顺序运行。例如，如果您为每个用户创建一个池，这意味着每个用户将获得集群的同等份额，并且每个用户的查询将按顺序运行，而不是以后的查询从该用户以前的查询中获取资源。

#### 配置pool属性

可以通过配置文件来修改pool的配置。每个pool都有3个属性：

- `schedulingMode`

  FIFO或FAIR。控制pool中的作业是按顺序排队，还是公平地共享pool的资源。

- `weight`

  默认情况下，所有pool的权重均为1。如果给某pool的权重设置为2，则它将获得比其他pool多1倍的资源。设置高权重的pool（如1000）可以实现pool之间的优先级。高优先级的pool中的活跃job有限启动。

- `minShare`

  每个pool的最小资源量。每个pool都可以获得设置的最小资源量（CPU核心数）。FAIR调度会满足所有活跃的pool的最小资源量。默认情况下，每个pool的`minShare`为0.。

pool的属性可以通过`.xml`文件来配置，如`conf/fairscheduler.xml`。或者通过`spark.scheduler.allocation.file`来设置配置文件所在路径

```scala
conf.set("spark.scheduler.allocation.file", "/path/to/file")
```

配置文件格式如下：

```xml
<?xml version="1.0"?>
<allocations>
  <pool name="production">
    <schedulingMode>FAIR</schedulingMode>
    <weight>1</weight>
    <minShare>2</minShare>
  </pool>
  <pool name="test">
    <schedulingMode>FIFO</schedulingMode>
    <weight>2</weight>
    <minShare>3</minShare>
  </pool>
</allocations>
```

***注意：如果所有的pool都没有做配置的话，就使用模式的配置值（schedulingMode=FIFO，weight=1，minShare=0）***

要为JDBC客户端会话设置公平调度程序池，用户可以设置spark.sql.thriftserver.Scheduler.pool变量

```sql
SET spark.sql.thriftserver.scheduler.pool=accounting;

```



## 1.15 Spark相关的硬件资源配置

参考：https://spark.apache.org/docs/3.0.2/hardware-provisioning.html

- 存储系统

  通常Spark作业都会从外部存储系统（）读取数据，作为作业的输入。数据节点和计算节点的距离越近，读取数据的效率越高。

  - Spark计算节点和数据节点为同一节点

  - Spark计算节点和数据节点为同一局域网中的不同节点

    某些情况下，为避免数据节点资源和计算节点资源使用抢占，可以分别安装在不同节点（如云厂商的存算分离架构）

- 本地磁盘

  建议4-8块盘（也有12块盘的，盘多的话可以保证不受单块盘磁盘IO的限制，提高磁盘IO的效率），不要配置RAID（每块盘单独挂载点）。`Spark.local.dir配置spark`的本地磁盘目录列表。

- 内存

  一般分配75%的内存用于Spark计算，剩下的内存给OS和buffer cache使用。通过Spark应用程序的监控UI（`http:<driver-node>:4040`）可以查看应用程序的数据集大小。

  内存使用率受到Spark的存储级别和序列化格式的影响。

  另外，建议分配给一个JVM的内存不要超过200GB，200GB内存的JVM 表现并不好。而是启动多个JVM（多个executor）这样性能更好。

- 网络

  一般10Gbit够用了，提高网络带宽可以有效提高应用程序的执行效率（提高了shuffle之前的网络传输，其他方面的网络传输等）。

  通过Spark应用程序的监控UI（`http:<driver-node>:4040`）可以查看spark shuffle的情况。

- CPU

  一般8-16个cpu核心，具体还要看工作负载，CPU越多可以有更高的并发度。



# 2. 安装spark

## 2.1 Windows10 安装Spark本地开发环境

参考文档：

[Spark在Win10下的环境搭建](https://blog.csdn.net/songhaifengshuaige/article/details/79480491)

[【Spark笔记】Windows10 本地搭建单机版Spark开发环境](https://www.shuzhiduo.com/A/n2d92KLYzD/)

### 2.1.1 **版本说明**

- JDK 1.8
- Scala 2.12.x，具体版本为2.12.17
- Hadoop 3.2.x
- IntelliJ IDEA 2019.3.3 (Ultimate Edition)
- Spark 3.0.2

Spark版本和Hadoop的版本关系并没有明确的限制，但是Spark官方已经预编译好了几个版本的spark包，我们可以直接下载使用，如果预编译好的spark包中没有合适，则需要自己来通过spark的源码进行编译了。

![image-20230128152245379](Spark.assets/image-20230128152245379.png)

Spark 3.0.2的官方下载地址：https://archive.apache.org/dist/spark/spark-3.0.2/

当下载spark-3.0.2时发现，有如下几种spark包：

- SparkR_3.0.2.tar.gz：[SparkR](http://www.iteblog.com/archives/tag/sparkr)是一个R语言包，它提供了轻量级的方式使得可以在R语言中使用Apache [Spark](http://www.iteblog.com/archives/tag/spark)。包括SparkR 的交互式命令行窗口。
- pyspark-3.0.2.tar.gz：PySpark简单来说就是[Spark](https://so.csdn.net/so/search?q=Spark&spm=1001.2101.3001.7020)提供的Python编程API，包括交互式的PySpark shell和非交互式的Python程序。参考：https://blog.csdn.net/qq_40856560/article/details/116027896
- spark-3.0.2-bin-hadoop2.7-hive1.2.tgz： 预编译好的支持Hadoop 2.7.x，Hive 1.2.x的spark 3.0.2的包
- spark-3.0.2-bin-hadoop2.7.tgz：  预编译好的支持Hadoop 2.7.x 的spark 3.0.2的包
- spark-3.0.2-bin-hadoop3.2.tgz： 预编译好的支持Hadoop 3.2.x 的spark 3.0.2的包
- spark-3.0.2-bin-without-hadoop.tgz：  spark中没有关联Hadoop的包，当运行此版本的spark时，需要用户手动将spark和hadoop关联上，参考：https://blog.csdn.net/yoshubom/article/details/104598483
- spark-3.0.2.tgz：  spark源码包，可以根据需要自己编译出指定Hadoop版本，Hive版本的spark包



### 2.1.2 **环境准备**

#### 2.1.2.1 JDK 安装和配置

##### 2.1.2.1.1 JDK下载

JDK下载地址：https://www.oracle.com/java/technologies/downloads/

目前最新的稳定版为JDK 17

![image-20230118174038282](Spark.assets/image-20230118174038282.png)

但是，我们要下载JDK8。单击页面中的`Java archive`

![image-20230118174153711](Spark.assets/image-20230118174153711.png)

单击Java SE 8，进入JDK 8 下载页面。

![image-20230118174211002](Spark.assets/image-20230118174211002.png)

单击下载[jdk-8u271-windows-x64.exe](https://www.oracle.com/java/technologies/javase/javase8u211-later-archive-downloads.html#license-lightbox)

![image-20230118174536960](Spark.assets/image-20230118174536960.png)



##### 2.1.2.1.2 JDK安装

双击`jdk-8u271-windows-x64.exe`进行JDK8的安装。

##### 2.1.2.1.3 JDK配置

在系统环境变量中添加`JAVA_HOME`，并将`JAVA_HOME`添加到系统环境变量的Path中。

增加 `JAVA_HOME`：D:\services\Java\jdk1.8.0_261

在`Path`中新增：%JAVA_HOME%\bin和%JAVA_HOME%\jre\bin

`CLASSPATH`中新增：%JAVA_HOME%/lib/dt.jar;%JAVA_HOME%/lib/tools.jar

验证JDK是否安装正确了

![image-20230118175427528](Spark.assets/image-20230118175427528.png)

说明JDK已经安装好了。

#### 2.1.2.2 Scala 安装和配置

[Scala官网](https://www.scala-lang.org/)

##### 2.1.2.2.1 Scala下载

Scala下载地址：https://www.scala-lang.org/download/all.html

![image-20230118162630649](Spark.assets/image-20230118162630649.png)

下载scala-2.12.17.zip，，然后在手动配置系统环境变量即可。当然也可以下载scala 2.12.17.msi，然后根据提示进行安装

![image-20230118171611781](Spark.assets/image-20230118171611781.png)

![image-20230118171534372](Spark.assets/image-20230118171534372.png)

##### 2.1.2.2.2 Scala安装

解压`scala-2.12.17.zip`到`D:\services\scala-2.12.17`

##### 2.1.2.2.3 Scala配置

在系统环境变量中添加`SCALA_HOME`，并将`SCALA_HOME`添加到系统环境变量的Path中。

**我的电脑 > 属性 > 高级系统配置 > 环境变量**

进入**高级系统设置**

![image-20230118160120400](Spark.assets/image-20230118160120400.png)

进入**环境变量**

![image-20230118160244021](Spark.assets/image-20230118160244021.png)

新建`SCALA_HOME`环境变量

![image-20230118160326294](Spark.assets/image-20230118160326294.png)

![image-20230118172013866](Spark.assets/image-20230118172013866.png)

将 `SCALA_HOME` 添加到系统环境变量Path中

![image-20230118160419853](Spark.assets/image-20230118160419853.png)

![image-20230118172125080](Spark.assets/image-20230118172125080.png)

![image-20230118160825198](Spark.assets/image-20230118160825198.png)

![image-20230118160957612](Spark.assets/image-20230118160957612.png)

验证下scala是否安装和配置正确了

![image-20230118172614227](Spark.assets/image-20230118172614227.png)

上图说明scala安装好了。

#### 2.1.2.3 Hadoop 安装和配置

##### 2.1.2.3.1 Hadoop下载

浏览器打开https://hadoop.apache.org/releases.html 页面，点击图中的连接（https://archive.apache.org/dist/hadoop/common），跳转到历史版本的hadoop的下载页面。

![image-20230128163307457](Spark.assets/image-20230128163307457.png)

找到hadoop-3.2.2，点击进入。

![image-20230128163405675](Spark.assets/image-20230128163405675.png)

可以看到，有好几种hadoop的tar包。

![image-20230128163619509](Spark.assets/image-20230128163619509.png)



- hadoop-3.2.2-rat.txt：
- hadoop-3.2.2-site.tar.gz：
- hadoop-3.2.2-src.tar.gz： 
- hadoop-3.2.2.tar.gz：

这里选择下载hadoop-3.2.2.tar.gz，但是从官网下载十分慢，所以在华为云镜像网站进行下载，网址如下：https://repo.huaweicloud.com/apache/hadoop/core/hadoop-3.2.2/

![image-20230128164926289](Spark.assets/image-20230128164926289.png)

下载完毕。

![image-20230128170106000](Spark.assets/image-20230128170106000.png)



##### 2.1.2.3.2 Hadoop安装

解压Hadoop安装包，解压到`D:\services\`

![image-20230128170554976](Spark.assets/image-20230128170554976.png)



##### 2.1.2.3.3 Hadoop配置

在系统环境变量中添加`HADOOP_HOME`，并将`HADOOP_HOME`添加到系统环境变量的Path中。

**我的电脑 > 属性 > 高级系统配置 > 环境变量**

进入**高级系统设置**

![image-20230118160120400](Spark.assets/image-20230118160120400.png)

进入**环境变量**

![image-20230118160244021](Spark.assets/image-20230118160244021.png)

新建`HADOOP_HOME`环境变量

![image-20230118160326294](Spark.assets/image-20230118160326294.png)



![image-20230128172202780](Spark.assets/image-20230128172202780.png)

将 `HADOOP_HOME` 添加到系统环境变量Path中

![image-20230118160419853](Spark.assets/image-20230118160419853.png)



![image-20230128172317752](Spark.assets/image-20230128172317752.png)

![image-20230118160825198](Spark.assets/image-20230118160825198.png)

![image-20230118160957612](Spark.assets/image-20230118160957612.png)

验证Hadoop的环境变量是否配置好了。

![image-20230128174017896](Spark.assets/image-20230128174017896.png)

执行`hadoop version`命令是正常的。

#### 2.1.2.4 Spark 安装和配置

##### 2.1.2.4.1 Spark下载

[Spark官方网站下载地址](https://spark.apache.org/downloads.html)

![image-20230118154016780](Spark.assets/image-20230118154016780.png)

*<font color="red">注意：Spark 3是由Scala 2.12构建的，Spark 3.2+是由Scala 2.13构建的，在安装Spark时需要安装对应版本的Scala。</font>*

由于当前Spark最新的稳定版为`spark-3.3.1-bin-hadoop3.tgz`，因此我们需要找到之前版本的spark。

![image-20230118154221456](Spark.assets/image-20230118154221456.png)

单击`archived releases`中的网址，如下所示：

![image-20230118154306725](Spark.assets/image-20230118154306725.png)

找到`spark-3.0.2`的目录，进去之后单击下载`spark-3.0.2-bin-hadoop3.2.tgz`。

![image-20230118154839649](Spark.assets/image-20230118154839649.png)

##### 2.1.2.4.2 Spark安装

解压spark安装包，解压到`D:\services\spark-3.0.2-bin-hadoop3.2`

![image-20230118155343855](Spark.assets/image-20230118155343855.png)

##### 2.1.2.4.3 Spark配置

在系统环境变量中添加`SPARK_HOME`，并将`SPARK_HOME`添加到系统环境变量的Path中。

**我的电脑 > 属性 > 高级系统配置 > 环境变量**

进入**高级系统设置**

![image-20230118160120400](Spark.assets/image-20230118160120400.png)

进入**环境变量**

![image-20230118160244021](Spark.assets/image-20230118160244021.png)

新建`SPARK_HOME`环境变量

![image-20230118160326294](Spark.assets/image-20230118160326294.png)



![image-20230118160355760](Spark.assets/image-20230118160355760.png)

将 `SPARK_HOME` 添加到系统环境变量Path中

![image-20230118160419853](Spark.assets/image-20230118160419853.png)



![image-20230118160529498](Spark.assets/image-20230118160529498.png)

![image-20230118160825198](Spark.assets/image-20230118160825198.png)

![image-20230118160957612](Spark.assets/image-20230118160957612.png)

验证Spark的环境变量是否配置好了。

![image-20230128171847033](Spark.assets/image-20230128171847033.png)

如果出现如上图的提示HADOOP_HOME没有配置，说明`HADOOP_HOME`的环境变量配置有问题，需要正确配置`HADOOP_HOME`。

正确配置好`HADOOP_HOME`之后，执行`spark-shell`命令。

![image-20230128174253841](Spark.assets/image-20230128174253841.png)

虽然最终进入到了spark shell中，但是中间报了一个错误，提示找不到`D:\services\hadoop-3.2.2\bin\winutils.exe`文件，通过查看发现确实不存在该文件，此时我们需要从https://github.com/srccodes/hadoop-common-2.2.0-bin/tree/master/bin此处下载`winutils.exe`文件，并保存到本地`D:\services\hadoop-3.2.2\bin\`目录下。然后再次运行`spark-shell`，结果如下：

![image-20230128175426534](Spark.assets/image-20230128175426534.png)

`spark-shell`启动正常了。

浏览器访问 http://127.0.0.1:4041/jobs 可以打开spark web ui。如下图所示：

![image-20230128180603406](Spark.assets/image-20230128180603406.png)





#### 2.1.2.5 Idea Scala插件安装

##### 2.1.2.5.1 常用插件

- Big Data Tools

  Big Data Tools插件的作用

  - Zeppelin笔记本中的探索性分析、可视化和原型制作工作。
  - 直接从IDE运行和监视Spark或Flink作业。
  - 使用Amazon EMR集群。
  - 查看大数据二进制文件，如CSV、Parquet、ORC和Avro。
  - 使用Kafka制作和消费消息。
  - 预览Hive Metastore数据库。
  - 了解Hadoop环境。

  参考文档：https://blog.csdn.net/xianpanjia4616/article/details/124938248

  ![image-20230129161527124](Spark.assets/image-20230129161527124.png)

- Scala

  添加对Scala语言的支持。IntelliJ IDEA社区版免费提供以下功能：

  编码帮助（突出显示、完成、格式化、重构等）

  导航、搜索、关于类型和隐式的信息

  与sbt和其他构建工具集成

  测试框架支持（ScalaTest、Specs2、uTest）

  Scala调试器、工作表和Ammonite脚本

  ![image-20230129162010952](Spark.assets/image-20230129162010952.png)

- Maven Helper

  分析和排除冲突依赖关系的简单方法
  为包含当前文件的模块或根模块运行/调试maven目标的操作
  在当前maven模块路径上打开终端的操作
  运行/调试当前测试文件的操作。

  ![image-20230129162325405](Spark.assets/image-20230129162325405.png)

- 2

- 

##### 2.1.2.5.2 在IDEA中如何创建scala.class文件

参考：https://blog.csdn.net/u010416101/article/details/127505064

添加本地的scala 地址
`Project Structure --> Global Libraries -> 添加`

加入后, 你可以发现IDEA已经能加载scala Library了.

# 3. spark常用命令



spark自带的示例

```shell
# For Scala and Java, use run-example:
./bin/run-example SparkPi

# For Python examples, use spark-submit directly:
./bin/spark-submit examples/src/main/python/pi.py

# For R examples, use spark-submit directly:
./bin/spark-submit examples/src/main/r/dataframe.R
```



# 4. spark配置

TODO

# 5. Spark迁移

参考：https://spark.apache.org/docs/3.0.2/migration-guide.html

## 5.1 Spark Core

假设Spark Core 2.4 迁移/升级为 Spark Core 3.0

- 因为某些接口有变化，对应的Spark应用程序可能需要重写代码，并编译。如 `org.apache.spark.ExecutorPlugin` 接口被替换为`org.apache.spark.api.plugin.SparkPlugin`。
- 旧版本的某些方法可以已经弃用，或者删除了，对应的Spark应用程序可能需要重写代码，并编译。如`shuffleBytesWritten`, `shuffleWriteTime` 和 `shuffleRecordsWritten` 

## 5.2 SQL, Datasets and DataFrame

pass

## 5.3 Structured Streaming

pass

## 5.4 MLlib (Machine Learning)

pass

## 5.5 PySpark (Python on Spark)

pass

## 5.6 SparkR (R on Spark)

pass