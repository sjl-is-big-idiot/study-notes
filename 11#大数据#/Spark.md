# 1. spark概述



<font color="red">***注意：Spark3.0、Spark3.1是由Scala2.12预编译，但是Spark 3.2+是采用Scala 2.13预编译。***</font>

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

# 5. spark常见问题

# 6

***注意，在Spark 2.0之前，Spark的主要编程接口是弹性分布式数据集（RDD）。在Spark 2.0之后，RDD被Dataset所取代，Dataset与RDD相似，但在后台进行了更丰富的优化。RDD接口仍然受支持，但DataSet比RDD性能更好。***

https://spark.apache.org/docs/3.0.2/rdd-programming-guide.html

https://spark.apache.org/docs/3.0.2/sql-programming-guide.html



注意，默认情况下Spark的security是关闭的。这意味着容易受到攻击。参考：https://spark.apache.org/docs/3.0.2/security.html

## 交互式命令行

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



## 应用程序调用Spark API

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

报错提示找不到主类SimpleApp。解决方案：主类写为全路径"org.spark.sjl.SimpleApp"。



## RDD编程

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