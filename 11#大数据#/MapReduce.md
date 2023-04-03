# 1. MapReduce概述

## 1.1 MapReduce定义

![image-20210912205222426](MapReduce.assets/image-20210912205222426.png)

参考文档：https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html

mapreduce job的Input和Output都存储在file system中。

MR框架负责**调度任务**、**监控任务**、**重新执行失败的任务**。MRAppMaster

MR作业需要指定三类信息：

- Input/Output的位置
- map和reduce函数（通过实现Mapper和Reducer接口来实现）
- 其他的Job配置参数

可以通过`Hadoop Job client`来提交Job（jar或者其他可执行文件）

虽然Hadoop框架是由Java实现的，但是MR Job不是一定要用Java来写的。

- `Hadoop Streaming` ：用可执行文件方式来创建/运行Job。
- `Hadoop Pipes` ：是一个SWIG，兼容C++ API来实现MR application

MR application操作的是<key, value>对，输入是<key, value>对，输出也是<key, value>对。因为key/value会被序列化，因此key、value的类需要实现`Writable`接口。key类需要实现`WritableComparable`接口，以便可以按key来排序。

**mapper**

mapper是MR框架中的一个接口。

mapper将Input的key/value对映射为中间结果的key/value对。MR框架用InputFormat生成InputSplit，为每个InputSplit生成一个map任务。`Job.setMapperClass(class)`可以将mapper的实现类传递给Job。然后在每个InputSplit中的每个key/value对调用map函数。map的输出的key/value对的类型可以不与输入的key/value对的类型一致。

sort -> partition -> combine -> spill -> compress

**map的数量？**

map的数量跟input的字节数、input的文件的block数量有关系。因为会为每个InputSplit产生一个map任务，因此需要看如何划分InputSplit。通常一个node可以有10-100个map，针对cpu计算较少的job可以为一个node设置300个map。建议一个map运行时间在1min以上（因为创建和销毁map也需要开销，如果创建出来就运行几秒钟，那更多的时间浪费在了创建/销毁map了，那么可以考虑减少map数量，而增加每个map处理的数据量）。

`set(MRJobConfig.NUM_MAPS, int)`只是为MR框架建议一个map的值，不是设置之后，map的数量就一定是这个值。

**reducer**

reducer对map的输出结果进行聚合。reducer是MR框架中的一个接口。

reduce的个数可以通过`Job.setNumReducerTasks(Int)`来设置。通过Job.setReducerClass(class)将Reducer的实现类传递给Job。MR框架调用reduce函数来处理每一个key/value对。

reducer有三个主要阶段：shuffle、sort、reduce。

- shuffle：传递给reduce的input是已经排序的map的输出。MR框架ton过HTTP获取每个map中由对应reduce处理的partition。

- sort：sort和shuffle同时发生，获取到map的输出结果之后还会进行merge。

- 二次sort：在shuffle时是按照key进行排序的，但是某些情况还需要对value中的每个字段来进行排序，这就是二次排序。

  二次排序的实现有两种方式：

  - 将此key的所有value都获取到后，全部缓存到内存进行排序。缺点是数据量不能太大。
  - 将key，value作为新的key，如key-value，让框架对新的key进行排序。

- reduce：为每个key/value对调用reduce函数，reduce的输出是未排序的。

**reduce的数量**

建议值： 0.95或1.75 * 节点数 * 节点的最大container数。

reduce的个数可以为0个，比如sqoop的任务就是只有map没有reduce的。当reduce为0时，map直接将结果写入输出路径，且不会进行排序（因为没有聚合，无法做到全局有序）。

**Partitioner**

partitioner控制中间结果（map输出）的key分配到哪个partition，在shuffle copy时被发送到哪个reduce。通常按Hash算法来分区，**分区数量=reduce任务的数量**。`Hash Partitioner`是默认的Partitioner。

**Counter**

Counter是用来报告application统计信息的一个工具。Mapper和Reducer的实现类可以通过Counter来汇报统计信息。

**Job的配置文件**

MR框架中有一个Job接口，可以让用户来描述一个MR Job。

**Task执行和环境**

task继承`MRAppMaster`的环境。可以通过`apreduce.[map | reduce].java.opts`来指定task的jvm参数。

**内存管理**

可以通过`mapreduce.[map | reduce].memory.mb`来设置每个map/reduce的最大内存，必须大于等于`-Xmx`设置的JVM内存。通常`-Xmx`此值的80%。

**InputSplit**

一个InputSplit表示将由一个mapper来处理的数据。是一个逻辑上的实例。

**RecordReader**

负责从一个InputSplit读取<key, value>对。

**InputFormat**

描述了MapReduce作业的输入规范。
MapReduce框架依赖作业的InputFormat：

- 将输入文件拆分为逻辑InputSplit实例，然后将每个InputSplit实例分配给一个mapper。
- 提供RecordReader实现，用于从逻辑InputSplit中收集输入记录以供mapper处理。

**OutputFormat**

描述了MapReduce作业的输出规范。
MapReduce框架依赖作业的OutputFormat：

- 验证作业的输出规范；例如，检查输出目录是否不存在。
- 提供用于写入作业输出文件的RecordWriter实现。输出文件存储在文件系统中。

**OutputCommitter**

TODO

**RecordWriter**

RecordWriter 负责将输出结果的 `<key, value>`对写入文件中。

**Counters**

Counter是由MR框架或application定义的全局计数器。

**DistributedCache（分布式缓存）**

`DistributedCache`分发特定任务的、大的、只读的文件。

`DistributedCache`是MapReduce框架的一个工具，作用是缓存application的文件（如 text，archive，jar等）。

可以在Job中指定需要缓存的文件（文件需要已经存在于hdfs上了）。

在task开始执行之前，MR框架会拷贝必要的文件到worker节点。

`DistributedCache`会跟踪缓存文件的修改时间。在task执行过程中，缓存的文件不应该被修改，修改后就需要重新获取文件了，降低效率，且容易导致多个task获取到的缓存文件内容不一致，出现问题。

`DistributedCache`可以分发简单的、只读的数据/文件，也可以分发archive（如zip，tar，tgz，tar.gz）和jar。

缓存file和archive

> `mapreduce.job.cache.{files |archives}`，或`Job.addCacheFile(URI)`/`Job.addCacheArchive(URI)` and `Job.setCacheFiles(URI[\])`/ `Job.setCacheArchives(URI[\])`可以设置需要缓存的file、archive。`Job.addArchiveToClassPath(Path)`，或`Job.addFileToClassPath(Path)`或`mapreduce.job.classpath.{files |archives}`可以将文件/archive/jar添加到task的jvm classpath中。

分布式缓存的文件可以是private或public的。这决定了缓存的文件在worker节点上如何共享。

- private的分布式缓存文件

  这类文件缓存在本地目录中。这类分布式缓存文件仅由提交此作业的用户所共享，此用户的所有job可以共享这些文件，其他用户和job不能访问。

- public的分布式缓存文件

  这类文件缓存在全局目录中。所有用户和job都可以使用。

**加密shuffle**

因为未加密时，shuffle是基于HTTP协议进行通信的，所以加密shuffle是基于HTTPS协议进行通信的。

**支持YARN共享缓存**

MR吃吃YARN的共享缓存，通过此种机制可以利用其他资源（不用每个application都分发此类资源）。这样可以减少Job提交机和YARN集群之间的网络IO，减少Job提交时间和总体的Job运行时间。

YARN集群运行了shared cache服务。

`mapred-site.xml`中的`mapreduce.job.sharedcache.mode`配置参数控制哪些类型的资源可以上传到YARN的共享缓存。

```xml
<property>
    <name>mapreduce.job.sharedcache.mode</name>
    <value>disabled</value>
    <description>
       A comma delimited list of resource categories to submit to the
       shared cache. The valid categories are: jobjar, libjars, files,
       archives. If "disabled" is specified then the job submission code
       will not use the shared cache.
    </description>
</property>
```



MR有三种方式可以指定MR job需要的资源：

- 命令行方式

  如`-files`, `-archives`, `-libjars`。如果资源的类型匹配，且MR配置开启了shared cache，那么这些配置指定的资源在类型匹配的时候会先去shared cache中找。

- distributed cache api

  如果指定了`distributed cache`，不管shared cache是否启用，都直接走distributed cache。

- shared cache api



**向后兼容**

向后兼容表示向低版本兼容。

**map相关参数**

```bash
mapreduce.task.io.sort.mb							# 环形缓冲区的大小
mapreduce.map.sort.spill.percent			# 一旦环形buffer达此阈值，就会安排一个线程将环形buffer中的内容spill到磁盘
																			# 如果在spill的过程中，环形buffer又满了，会再启动线程来spill，前一此spill
																			# 不会阻塞下一次spill
																			# 大于环形buffer的记录会先触发一次spill，spill到一个单独的文件				
```

**shuffle/reduce相关参数**

```bash
mapreduce.task.io.soft.factor										# 控制merge过程同时打开的segment数量。默认是10，
mapreduce.reduce.merge.inmem.thresholds					# 默认值1000，表示开始spill的map输出文件数阈值，<=0表示没有阈值，
																								#	此时只由缓冲池比例来控制。
mapreduce.reduce.shuffle.merge.percent					# 默认值0.66，表示开始spill的缓冲池的比例阈值，达到0.66开始spill。
mapreduce.reduce.shuffle.input.buffer.percent		# 默认值0.7，表示shuffle copy阶段用于保存map输出的堆内存的比例
																								# 相对mapreduce.reduce.java.opts的比例
mapreduce.reduce.input.buffer.percent						# reduce期间用来缓存map结果大小，堆内存百分比。当shuflle结束后，
																								# 内存中剩余数据必须小于此值才开始reduce计算。默认情况下map输出都写入磁盘
mapreduce.reduce.shuffle.parallelcopies					# 默认值5，表示拉取map输出结果的copier线程数。
```



查看mapreduce的历史日志

```bash
mapred job -history output.jhist
```



## 1.2 MapReduce优缺点

### 1.2.1 优点

![img](MapReduce.assets/wps1.png)

![img](MapReduce.assets/wps2.png)

### 1.2.2 缺点

![img](MapReduce.assets/wps3.png)

### 1.3 MapReduce核心思想

MapReduce核心编程思想，如下图所示。

![img](MapReduce.assets/wps4.png)

1. 分布式的运算程序往往需要分成至少2个阶段。
2. 第一个阶段的MapTask并发实例，完全并行运行，互不相干。
3. 第二个阶段的ReduceTask并发实例互不相干，但是他们的数据依赖于上一个阶段的所有MapTask并发实例的输出。
4. MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段，如果用户的业务逻辑非常复杂，那就只能多个MapReduce程序，串行运行。

总结：分析WordCount数据流走向深入理解MapReduce核心思想。

## 1.4 MapReduce进程

![img](MapReduce.assets/wps5.png)

## 1.5 官方WordCount源码

采用反编译工具反编译源码，发现WordCount案例有Map类、Reduce类和驱动类。且数据的类型是Hadoop自身封装的序列化类型。

## 1.6 常用数据序列化类型

| **Java类型** | **Hadoop Writable类型** |
| ------------ | ----------------------- |
| boolean      | BooleanWritable         |
| byte         | ByteWritable            |
| int          | IntWritable             |
| float        | FloatWritable           |
| long         | LongWritable            |
| double       | DoubleWritable          |
| String       | Text                    |
| map          | MapWritable             |
| array        | ArrayWritable           |

## 1.7 MapReduce编程规范

用户编写的程序分成三个部分：Mapper、Reducer和Driver。

![img](MapReduce.assets/wps6.png)

![img](MapReduce.assets/wps7.png)

## 1.8 WordCount案例实操

1. 需求

   在给定的文本文件中统计输出每一个单词出现的总次数

   （1）输入数据

   ![img](MapReduce.assets/wps8.png)

   （2）期望输出数据

   atguigu	2

   banzhang	1

   cls	2

   hadoop	1

   jiao	1

   ss	2

   xue	1

2. 需求分析

   按照MapReduce编程规范，分别编写Mapper，Reducer，Driver，如下图所示。

   ![img](MapReduce.assets/wps9.png)

3. 环境准备

   （1）创建maven工程

   ![img](MapReduce.assets/wps10.jpg) 

   ![img](MapReduce.assets/wps11.jpg) 

   ![img](MapReduce.assets/wps12.jpg) 

   ![img](MapReduce.assets/wps13.jpg) 

   （2）在pom.xml文件中添加如下依赖

   ```xml
   <dependencies>
   		<dependency>
   			<groupId>junit</groupId>
   			<artifactId>junit</artifactId>
   			<version>RELEASE</version>
   		</dependency>
   		<dependency>
   			<groupId>org.apache.logging.log4j</groupId>
   			<artifactId>log4j-core</artifactId>
   			<version>2.8.2</version>
   		</dependency>
   		<dependency>
   			<groupId>org.apache.hadoop</groupId>
   			<artifactId>hadoop-common</artifactId>
   			<version>2.7.2</version>
       	</dependency>
   		<dependency>
   			<groupId>org.apache.hadoop</groupId>
   			<artifactId>hadoop-client</artifactId>
   			<version>2.7.2</version>
   		</dependency>
   		<dependency>
   			<groupId>org.apache.hadoop</groupId>
   			<artifactId>hadoop-hdfs</artifactId>
   			<version>2.7.2</version>
   		</dependency>
   </dependencies>
   ```

   

   （2）在项目的src/main/resources目录下，新建一个文件，命名为“log4j.properties”，在文件中填入。

   ```properties
   log4j.rootLogger=INFO, stdoutlog4j.appender.stdout=org.apache.log4j.ConsoleAppenderlog4j.appender.stdout.layout=org.apache.log4j.PatternLayoutlog4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%nlog4j.appender.logfile=org.apache.log4j.FileAppenderlog4j.appender.logfile.File=target/spring.loglog4j.appender.logfile.layout=org.apache.log4j.PatternLayoutlog4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
   ```



4. 编写程序

   （1）编写Mapper类

   ```java
   
   package com.atguigu.mapreduce;
   
   import java.io.IOException;
   
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class WordcountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
       Text k = new Text();
       IntWritable v = new IntWritable(1);
   
       @Override
       protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
           // 1 获取一行		
           String line = value.toString();
           // 2 切割		
           String[] words = line.split(" ");
           // 3 输出		
           for (String word : words) {
               k.set(word);
               context.write(k, v);
           }
       }
   }
   
   ```

   

   （2）编写Reducer类

   ```java
   package com.atguigu.mapreduce.wordcount;
   
   import java.io.IOException;
   
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class WordcountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
       int sum;
       IntWritable v = new IntWritable();
   
       @Override
       protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
           // 1 累加求和		
           sum = 0;
           for (IntWritable count : values) {
               sum += count.get();
           }
           // 2 输出    
           v.set(sum);
           context.write(key, v);
       }
   }
   
   ```

   （3）编写Driver驱动类

   ```java
   package com.atguigu.mapreduce.wordcount;
   
   import java.io.IOException;
   
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class WordcountDriver {
       public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
           // 1 获取配置信息以及封装任务		
           Configuration configuration = new Configuration();
           Job job = Job.getInstance(configuration);
           // 2 设置jar加载路径		
           job.setJarByClass(WordcountDriver.class);
           // 3 设置map和reduce类		
           job.setMapperClass(WordcountMapper.class);
           job.setReducerClass(WordcountReducer.class);
           // 4 设置map输出		
           job.setMapOutputKeyClass(Text.class);
           job.setMapOutputValueClass(IntWritable.class);
           // 5 设置最终输出kv类型		
           job.setOutputKeyClass(Text.class);
           job.setOutputValueClass(IntWritable.class);
           // 6 设置输入和输出路径		
           FileInputFormat.setInputPaths(job, new Path(args[0]));
           FileOutputFormat.setOutputPath(job, new Path(args[1]));
           // 7 提交		
           boolean result = job.waitForCompletion(true);
           System.exit(result ? 0 : 1);
       }
   }
   ```

   

5. 本地测试

   （1）如果电脑系统是win7的就将win7的hadoop jar包解压到非中文路径，并在Windows环境上配置HADOOP_HOME环境变量。如果是电脑win10操作系统，就解压win10的hadoop jar包，并配置HADOOP_HOME环境变量。

   注意：win8电脑和win10家庭版操作系统可能有问题，需要重新编译源码或者更改操作系统。

   ![img](MapReduce.assets/wps14.jpg) 

   （2）在Eclipse/Idea上运行程序

6. 集群上测试

   （0）用maven打jar包，需要添加的打包插件依赖

   注意：标记红颜色的部分需要替换为自己工程主类

   ```xml
   <build>
       <plugins>
           <plugin>
               <artifactId>maven-compiler-plugin</artifactId>
               <version>2.3.2</version>
               <configuration>
                   <source>1.8</source>
                   <target>1.8</target>
               </configuration>
           </plugin>
           <plugin>
               <artifactId>maven-assembly-plugin</artifactId>
               <configuration>
                   <descriptorRefs>
                       <descriptorRef>jar-with-dependencies</descriptorRef>
                   </descriptorRefs>
                   <archive>
                       <manifest>
                           <mainClass>com.atguigu.mr.WordcountDriver</mainClass>
                       </manifest>
                   </archive>
               </configuration>
               <executions>
                   <execution>
                       <id>make-assembly</id>
                       <phase>package</phase>
                       <goals>
                           <goal>single</goal>
                       </goals>
                   </execution>
               </executions>
           </plugin>
       </plugins>
   </build>
   ```

   注意：如果工程上显示红叉。在项目上右键->maven->update project即可。

   （1）将程序打成jar包，然后拷贝到Hadoop集群中

   步骤详情：右键->Run as->maven install。等待编译完成就会在项目的target文件夹中生成jar包。如果看不到。在项目上右键-》Refresh，即可看到。修改不带依赖的jar包名称为wc.jar，并拷贝该jar包到Hadoop集群。

   （2）启动Hadoop集群

   （3）执行WordCount程序

   ```shell
   [atguigu@hadoop102 software]$ hadoop jar  wc.jar
   
    com.atguigu.wordcount.WordcountDriver /user/atguigu/input /user/atguigu/output
   ```

   

# 2. Hadoop序列化

## 2.1 序列化概述

![img](MapReduce.assets/wps15.png)

![img](MapReduce.assets/wps16.png)

## 2.2 自定义bean对象实现序列化接口（Writable）

在企业开发中往往常用的基本序列化类型不能满足所有需求，比如在Hadoop框架内部传递一个bean对象，那么该对象就需要实现序列化接口。

具体实现bean对象序列化步骤如下7步。

1. 必须实现Writable接口

2. 反序列化时，需要反射调用空参构造函数，所以必须有空参构造

   ```java
   public FlowBean() {	super();}
   ```

3. 重写序列化方法

   ```java
   @Overridepublic void write(DataOutput out) throws IOException {	out.writeLong(upFlow);	out.writeLong(downFlow);	out.writeLong(sumFlow);}
   ```

   

4. 重写反序列化方法

   ```java
   @Override
   public void readFields(DataInput in) throws IOException {	
       upFlow = in.readLong();	
       downFlow = in.readLong();	
       sumFlow = in.readLong();
   }
   ```

5. <font color=red>注意反序列化的顺序和序列化的顺序完全一致</font>

6. 要想把结果显示在文件中，需要重写toString()，可用”\t”分开，方便后续用。

7. 如果需要将自定义的bean放在key中传输，则还需要实现Comparable接口，因为MapReduce框中的Shuffle过程要求对key必须能排序。详见后面排序案例。

   ```java
   @Override
   public int compareTo(FlowBean o) {	
       // 倒序排列，从大到小	
       return this.sumFlow > o.getSumFlow() ? -1 : 1;
   }
   ```

   

## 2.3 序列化案例实操

1. 需求

   统计每一个手机号耗费的总上行流量、下行流量、总流量

   （1）输入数据

   ![img](MapReduce.assets/wps17.png)

   （2）输入数据格式：

   | id   | 手机号码    | 网络ip         | 上行流量 | 下行流量 | 网络状态码 |
   | ---- | ----------- | -------------- | -------- | -------- | ---------- |
   | 7    | 13560436666 | 120.196.100.99 | 1116     | 954      | 200id      |

   （3）期望输出数据格式

   | 手机号码    | 上行流量 | 下行流量 | 网络状态码 | 总流量 |
   | ----------- | -------- | -------- | ---------- | ------ |
   | 13560436666 | 1116     | 954      | 200        | 2070   |

2. 需求分析

   ![img](MapReduce.assets/wps18.png)

3. 编写MapReduce程序

   （1）编写流量统计的Bean对象

   ```java
   package com.atguigu.mapreduce.flowsum;
   
   import java.io.DataInput;
   import java.io.DataOutput;
   import java.io.IOException;
   
   import org.apache.hadoop.io.Writable;
   
   // 1 实现writable接口
   public class FlowBean implements Writable {
   
       private long upFlow;
       private long downFlow;
       private long sumFlow;
   
       //2  反序列化时，需要反射调用空参构造函数，所以必须有
       public FlowBean() {
           super();
       }
   
       public FlowBean(long upFlow, long downFlow) {
           super();
           this.upFlow = upFlow;
           this.downFlow = downFlow;
           this.sumFlow = upFlow + downFlow;
       }
   
       //3  写序列化方法
       @Override
       public void write(DataOutput out) throws IOException {
           out.writeLong(upFlow);
           out.writeLong(downFlow);
           out.writeLong(sumFlow);
       }
   
       //4 反序列化方法
       //5 反序列化方法读顺序必须和写序列化方法的写顺序必须一致
       @Override
       public void readFields(DataInput in) throws IOException {
           this.upFlow = in.readLong();
           this.downFlow = in.readLong();
           this.sumFlow = in.readLong();
       }
   
       // 6 编写toString方法，方便后续打印到文本
       @Override
       public String toString() {
           return upFlow + "\t" + downFlow + "\t" + sumFlow;
       }
   
       public long getUpFlow() {
           return upFlow;
       }
   
       public void setUpFlow(long upFlow) {
           this.upFlow = upFlow;
       }
   
       public long getDownFlow() {
           return downFlow;
       }
   
       public void setDownFlow(long downFlow) {
           this.downFlow = downFlow;
       }
   
       public long getSumFlow() {
           return sumFlow;
       }
   
       public void setSumFlow(long sumFlow) {
           this.sumFlow = sumFlow;
       }
   }
   ```

   （2）编写Mapper类

   ```java
   package com.atguigu.mapreduce.flowsum;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class FlowCountMapper extends Mapper<LongWritable, Text, Text, FlowBean>{
   	
   	FlowBean v = new FlowBean();
   	Text k = new Text();
   	
   	@Override
   	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
   		
   		// 1 获取一行
   		String line = value.toString();
   		
   		// 2 切割字段
   		String[] fields = line.split("\t");
   		
   		// 3 封装对象
   		// 取出手机号码
   		String phoneNum = fields[1];
   
   		// 取出上行流量和下行流量
   		long upFlow = Long.parseLong(fields[fields.length - 3]);
   		long downFlow = Long.parseLong(fields[fields.length - 2]);
   
   		k.set(phoneNum);
   		v.set(downFlow, upFlow);
   		
   		// 4 写出
   		context.write(k, v);
   	}
   }
   ```

   （3）编写Reducer类

   ```java
   package com.atguigu.mapreduce.flowsum;
   import java.io.IOException;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class FlowCountReducer extends Reducer<Text, FlowBean, Text, FlowBean> {
   
   	@Override
   	protected void reduce(Text key, Iterable<FlowBean> values, Context context)throws IOException, InterruptedException {
   
   		long sum_upFlow = 0;
   		long sum_downFlow = 0;
   
   		// 1 遍历所用bean，将其中的上行流量，下行流量分别累加
   		for (FlowBean flowBean : values) {
   			sum_upFlow += flowBean.getUpFlow();
   			sum_downFlow += flowBean.getDownFlow();
   		}
   
   		// 2 封装对象
   		FlowBean resultBean = new FlowBean(sum_upFlow, sum_downFlow);
   		
   		// 3 写出
   		context.write(key, resultBean);
   	}
   }
   ```

   （4）编写Driver驱动类

   ```java
   package com.atguigu.mapreduce.flowsum;
   import java.io.IOException;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class FlowsumDriver {
   
   	public static void main(String[] args) throws IllegalArgumentException, IOException, ClassNotFoundException, InterruptedException {
   		
   // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
   args = new String[] { "e:/input/inputflow", "e:/output1" };
   
   		// 1 获取配置信息，或者job对象实例
   		Configuration configuration = new Configuration();
   		Job job = Job.getInstance(configuration);
   
   		// 6 指定本程序的jar包所在的本地路径
   		job.setJarByClass(FlowsumDriver.class);
   
   		// 2 指定本业务job要使用的mapper/Reducer业务类
   		job.setMapperClass(FlowCountMapper.class);
   		job.setReducerClass(FlowCountReducer.class);
   
   		// 3 指定mapper输出数据的kv类型
   		job.setMapOutputKeyClass(Text.class);
   		job.setMapOutputValueClass(FlowBean.class);
   
   		// 4 指定最终输出的数据的kv类型
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(FlowBean.class);
   		
   		// 5 指定job的输入原始文件所在目录
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
   		// 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
   		boolean result = job.waitForCompletion(true);
   		System.exit(result ? 0 : 1);
   	}
   }
   ```

   

# 3. MapReduce框架原理

## 3.1 InputFormat数据输入

### 3.1.1 切片与MapTask并行度决定机制

1. 问题引出

   MapTask的并行度决定Map阶段的任务处理并发度，进而影响到整个Job的处理速度。

   思考：1G的数据，启动8个MapTask，可以提高集群的并发处理能力。那么1K的数据，也启动8个MapTask，会提高集群性能吗？MapTask并行任务是否越多越好呢？哪些因素影响了MapTask并行度？![img](MapReduce.assets/wps19.jpg)

2. MapTask并行度决定机制

   **数据块：**Block是HDFS物理上把数据分成一块一块。

   **数据切片：**数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。

   ![img](MapReduce.assets/wps20.png)

### 3.1.2 Job提交流程源代码和切片源码详解

1. Job提交流程源码详解，如下图所示：

   ![img](MapReduce.assets/wps21.png)

   ```java
   waitForCompletion()
   
   submit();
   
   // 1建立连接
   	connect();	
   		// 1）创建提交Job的代理
   		new Cluster(getConfiguration());
   			// （1）判断是本地yarn还是远程
   			initialize(jobTrackAddr, conf); 
   
   // 2 提交job
   submitter.submitJobInternal(Job.this, cluster)
   	// 1）创建给集群提交数据的Stag路径
   	Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);
   
   	// 2）获取jobid ，并创建Job路径
   	JobID jobId = submitClient.getNewJobID();
   
   	// 3）拷贝jar包到集群
   copyAndConfigureFiles(job, submitJobDir);	
   	rUploader.uploadFiles(job, jobSubmitDir);
   
   // 4）计算切片，生成切片规划文件
   writeSplits(job, submitJobDir);
   		maps = writeNewSplits(job, jobSubmitDir);
   		input.getSplits(job);
   
   // 5）向Stag路径写XML配置文件
   writeConf(conf, submitJobFile);
   	conf.writeXml(out);
   
   // 6）提交Job,返回提交状态
   status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());
   ```

   

2. FileInputFormat切片源码解析(input.getSplits(job))

   ![img](MapReduce.assets/wps22.png)

### 3.1.3 FileInputFormat切片机制

![img](MapReduce.assets/wps23.png)

![img](MapReduce.assets/wps24.png)

### 3.1.4 CombineTextInputFormat切片机制

<font color=red>框架默认的TextInputFormat切片机制是对任务按文件规划切片，不管文件多小，都会是一个单独的切片，都会交给一个MapTask，这样如果有大量小文件，就会产生大量的MapTask，处理效率极其低下。</font>

1. 应用场景：

   CombineTextInputFormat用于小文件过多的场景，它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个MapTask处理。

2. 虚拟存储切片最大值设置

   CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m

   注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。

3. 切片机制

   生成切片过程包括：虚拟存储过程和切片过程二部分。

   ![img](MapReduce.assets/wps25.png)

### 3.1.5 CombineTextInputFormat案例实操

1. 需求

   将输入的大量小文件合并成一个切片统一处理。

   （1）输入数据

   准备4个小文件

   （2）期望

   期望一个切片处理4个文件

2. 实现过程

   （1）不做任何处理，运行1.6节的WordCount案例程序，观察切片个数为4。

   ![img](MapReduce.assets/wps26.jpg) 

   （2）在WordcountDriver中增加如下代码，运行程序，并观察运行的切片个数为3。

   ​	（a）驱动类中添加代码如下：

   ```java
   // 如果不设置InputFormat，它默认用的是TextInputFormat.class
   job.setInputFormatClass(CombineTextInputFormat.class);
   
    
   //虚拟存储切片最大值设置4m
   CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);
   ```

   

   ​	（b）运行如果为3个切片。

   ​		![img](MapReduce.assets/wps27.jpg) 

   （3）在WordcountDriver中增加如下代码，运行程序，并观察运行的切片个数为1。

   ​		（a）驱动中添加代码如下：

   ```java
   // 如果不设置InputFormat，它默认用的是TextInputFormat.class
   job.setInputFormatClass(CombineTextInputFormat.class);
   
   
   //虚拟存储切片最大值设置20m
   CombineTextInputFormat.setMaxInputSplitSize(job, 20971520);
   ```

   ​	（b）运行如果为1个切片。

   ​		![img](MapReduce.assets/wps28.jpg) 

### 3.1.6 FileInputFormat实现类

![img](MapReduce.assets/wps29.png)

![img](MapReduce.assets/wps30.png)

![img](MapReduce.assets/wps31.png)

![img](MapReduce.assets/wps32.png)

### 3.1.7 KeyValueTextInputFormat使用案例

1. 需求

   统计输入文件中每一行的第一个单词相同的行数。

   （1）输入数据

   banzhang ni hao

   xihuan hadoop banzhang

   banzhang ni hao

   xihuan hadoop banzhang

   （2）期望结果数据

   banzhang	2

   xihuan	2

2. 需求分析

   ![img](MapReduce.assets/wps33.png)

3. 代码实现

   （1）编写Mapper类

   ```java
   package com.atguigu.mapreduce.KeyValueTextInputFormat;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class KVTextMapper extends Mapper<Text, Text, Text, LongWritable>{
   	
   // 1 设置value
      LongWritable v = new LongWritable(1);  
       
   	@Override
   	protected void map(Text key, Text value, Context context)
   			throws IOException, InterruptedException {
   
   // banzhang ni hao
           
           // 2 写出
           context.write(key, v);  
   	}
   }
   ```

   （2）编写Reducer类

   ```java
   package com.atguigu.mapreduce.KeyValueTextInputFormat;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class KVTextReducer extends Reducer<Text, LongWritable, Text, LongWritable>{
   	
       LongWritable v = new LongWritable();  
       
   	@Override
   	protected void reduce(Text key, Iterable<LongWritable> values,	Context context) throws IOException, InterruptedException {
   		
   		 long sum = 0L;  
   
   		 // 1 汇总统计
           for (LongWritable value : values) {  
               sum += value.get();  
           }
            
           v.set(sum);  
            
           // 2 输出
           context.write(key, v);  
   	}
   }
   ```

   

   （3）编写Driver类

   ```java
   package com.atguigu.mapreduce.keyvaleTextInputFormat;
   import java.io.IOException;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.input.KeyValueLineRecordReader;
   import org.apache.hadoop.mapreduce.lib.input.KeyValueTextInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class KVTextDriver {
   
   	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
   		
   		Configuration conf = new Configuration();
   		// 设置切割符
   	conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR, " ");
   		// 1 获取job对象
   		Job job = Job.getInstance(conf);
   		
   		// 2 设置jar包位置，关联mapper和reducer
   		job.setJarByClass(KVTextDriver.class);
   		job.setMapperClass(KVTextMapper.class);
   job.setReducerClass(KVTextReducer.class);
   				
   		// 3 设置map输出kv类型
   		job.setMapOutputKeyClass(Text.class);
   		job.setMapOutputValueClass(LongWritable.class);
   
   		// 4 设置最终输出kv类型
   		job.setOutputKeyClass(Text.class);
   job.setOutputValueClass(LongWritable.class);
   		
   		// 5 设置输入输出数据路径
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		
   		// 设置输入格式
   	job.setInputFormatClass(KeyValueTextInputFormat.class);
   		
   		// 6 设置输出数据路径
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   		
   		// 7 提交job
   		job.waitForCompletion(true);
   	}
   }
   ```

   

### 3.1.8 NLineInputFormat使用案例

1. 需求

   对每个单词进行个数统计，要求根据每个输入文件的行数来规定输出多少个切片。此案例要求每三行放入一个切片中。

   （1）输入数据

   banzhang ni hao

   xihuan hadoop banzhang

   banzhang ni hao

   xihuan hadoop banzhang

   banzhang ni hao

   xihuan hadoop banzhang

   banzhang ni hao

   xihuan hadoop banzhang

   banzhang ni hao

   xihuan hadoop banzhang banzhang ni hao

   xihuan hadoop banzhang

   （2）期望输出数据

   Number of splits:4

2. 需求分析

   ![img](MapReduce.assets/wps35.png)

3. 代码实现

   （1）编写Mapper类

   ```java
   package com.atguigu.mapreduce.nline;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class NLineMapper extends Mapper<LongWritable, Text, Text, LongWritable>{
   	
   	private Text k = new Text();
   	private LongWritable v = new LongWritable(1);
   	
   	@Override
   	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
   		
   		 // 1 获取一行
           String line = value.toString();
           
           // 2 切割
           String[] splited = line.split(" ");
           
           // 3 循环写出
           for (int i = 0; i < splited.length; i++) {
           	
           	k.set(splited[i]);
           	
              context.write(k, v);
           }
   	}
   }
   ```

   

   （2）编写Redcuer类

   ```java
   package com.atguigu.mapreduce.nline;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class NLineReducer extends Reducer<Text, LongWritable, Text, LongWritable>{
   	
   	LongWritable v = new LongWritable();
   	
   	@Override
   	protected void reduce(Text key, Iterable<LongWritable> values,	Context context) throws IOException, InterruptedException {
   		
           long sum = 0l;
   
           // 1 汇总
           for (LongWritable value : values) {
               sum += value.get();
           }  
           
           v.set(sum);
           
           // 2 输出
           context.write(key, v);
   	}
   }
   ```

   

   （3）编写Driver类

   ```java
   package com.atguigu.mapreduce.nline;
   import java.io.IOException;
   import java.net.URISyntaxException;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.input.NLineInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class NLineDriver {
   	
   	public static void main(String[] args) throws IOException, URISyntaxException, ClassNotFoundException, InterruptedException {
   		
   // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
   args = new String[] { "e:/input/inputword", "e:/output1" };
   
   		 // 1 获取job对象
   		 Configuration configuration = new Configuration();
           Job job = Job.getInstance(configuration);
           
           // 7设置每个切片InputSplit中划分三条记录
           NLineInputFormat.setNumLinesPerSplit(job, 3);
             
           // 8使用NLineInputFormat处理记录数  
           job.setInputFormatClass(NLineInputFormat.class);  
             
           // 2设置jar包位置，关联mapper和reducer
           job.setJarByClass(NLineDriver.class);  
           job.setMapperClass(NLineMapper.class);  
           job.setReducerClass(NLineReducer.class);  
           
           // 3设置map输出kv类型
           job.setMapOutputKeyClass(Text.class);  
           job.setMapOutputValueClass(LongWritable.class);  
           
           // 4设置最终输出kv类型
           job.setOutputKeyClass(Text.class);  
           job.setOutputValueClass(LongWritable.class);  
             
           // 5设置输入输出数据路径
           FileInputFormat.setInputPaths(job, new Path(args[0]));  
           FileOutputFormat.setOutputPath(job, new Path(args[1]));  
             
           // 6提交job
           job.waitForCompletion(true);  
   	}
   }
   ```

4. 测试

   （1）输入数据

   banzhang ni hao

   xihuan hadoop banzhang

   banzhang ni hao

   xihuan hadoop banzhang

   banzhang ni hao

   xihuan hadoop banzhang

   banzhang ni hao

   xihuan hadoop banzhang

   banzhang ni hao

   xihuan hadoop banzhang banzhang ni hao

   xihuan hadoop banzhang

（2）输出结果的切片数，如下图所示：

![image-20210913105401453](MapReduce.assets/image-20210913105401453.png)

### 3.1.9 自定义InputFormat

![img](MapReduce.assets/wps36.png)

### 3.1.10 自定义InputFormat案例实操

无论HDFS还是MapReduce，在处理小文件时效率都非常低，但又难免面临处理大量小文件的场景，此时，就需要有相应解决方案。可以自定义InputFormat实现小文件的合并。

1. 需求

   将多个小文件合并成一个SequenceFile文件（SequenceFile文件是Hadoop用来存储二进制形式的key-value对的文件格式），SequenceFile里面存储着多个文件，存储的形式为文件路径+名称为key，文件内容为value。

   （1）输入数据

   ![img](MapReduce.assets/wps37.png)   ![img](MapReduce.assets/wps38.png)   ![img](MapReduce.assets/wps39.png)

   （2）期望输出文件格式

   ![img](MapReduce.assets/wps40.png)

2. 需求分析

   ![img](MapReduce.assets/wps41.png)

3. 程序实现

   （1）自定义InputFromat

   ```java
   package com.atguigu.mapreduce.inputformat;
   import java.io.IOException;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.BytesWritable;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.mapreduce.InputSplit;
   import org.apache.hadoop.mapreduce.JobContext;
   import org.apache.hadoop.mapreduce.RecordReader;
   import org.apache.hadoop.mapreduce.TaskAttemptContext;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   
   // 定义类继承FileInputFormat
   public class WholeFileInputformat extends FileInputFormat<Text, BytesWritable>{
   	
   	@Override
   	protected boolean isSplitable(JobContext context, Path filename) {
   		return false;
   	}
   
   	@Override
   	public RecordReader<Text, BytesWritable> createRecordReader(InputSplit split, TaskAttemptContext context)	throws IOException, InterruptedException {
   		
   		WholeRecordReader recordReader = new WholeRecordReader();
   		recordReader.initialize(split, context);
   		
   		return recordReader;
   	}
   }
   ```

   

   （2）自定义RecordReader类

   ```java
   package com.atguigu.mapreduce.inputformat;
   import java.io.IOException;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.FSDataInputStream;
   import org.apache.hadoop.fs.FileSystem;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.BytesWritable;
   import org.apache.hadoop.io.IOUtils;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.mapreduce.InputSplit;
   import org.apache.hadoop.mapreduce.RecordReader;
   import org.apache.hadoop.mapreduce.TaskAttemptContext;
   import org.apache.hadoop.mapreduce.lib.input.FileSplit;
   
   public class WholeRecordReader extends RecordReader<Text, BytesWritable>{
   
   	private Configuration configuration;
   	private FileSplit split;
   	
   	private boolean isProgress= true;
   	private BytesWritable value = new BytesWritable();
   	private Text k = new Text();
   
   	@Override
   	public void initialize(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
   		
   		this.split = (FileSplit)split;
   		configuration = context.getConfiguration();
   	}
   
   	@Override
   	public boolean nextKeyValue() throws IOException, InterruptedException {
   		
   		if (isProgress) {
   
   			// 1 定义缓存区
   			byte[] contents = new byte[(int)split.getLength()];
   			
   			FileSystem fs = null;
   			FSDataInputStream fis = null;
   			
   			try {
   				// 2 获取文件系统
   				Path path = split.getPath();
   				fs = path.getFileSystem(configuration);
   				
   				// 3 读取数据
   				fis = fs.open(path);
   				
   				// 4 读取文件内容
   				IOUtils.readFully(fis, contents, 0, contents.length);
   				
   				// 5 输出文件内容
   				value.set(contents, 0, contents.length);
   
   // 6 获取文件路径及名称
   String name = split.getPath().toString();
   
   // 7 设置输出的key值
   k.set(name);
   
   			} catch (Exception e) {
   				
   			}finally {
   				IOUtils.closeStream(fis);
   			}
   			
   			isProgress = false;
   			
   			return true;
   		}
   		
   		return false;
   	}
   
   	@Override
   	public Text getCurrentKey() throws IOException, InterruptedException {
   		return k;
   	}
   
   	@Override
   	public BytesWritable getCurrentValue() throws IOException, InterruptedException {
   		return value;
   	}
   
   	@Override
   	public float getProgress() throws IOException, InterruptedException {
   		return 0;
   	}
   
   	@Override
   	public void close() throws IOException {
   	}
   }
   ```

   

   （3）编写SequenceFileMapper类处理流程

   ```java
   package com.atguigu.mapreduce.inputformat;
   import java.io.IOException;
   import org.apache.hadoop.io.BytesWritable;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   import org.apache.hadoop.mapreduce.lib.input.FileSplit;
   
   public class SequenceFileMapper extends Mapper<Text, BytesWritable, Text, BytesWritable>{
   	
   	@Override
   	protected void map(Text key, BytesWritable value,			Context context)		throws IOException, InterruptedException {
   
   		context.write(key, value);
   	}
   }
   ```

   

   （4）编写SequenceFileReducer类处理流程

   ```java
   package com.atguigu.mapreduce.inputformat;
   import java.io.IOException;
   import org.apache.hadoop.io.BytesWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class SequenceFileReducer extends Reducer<Text, BytesWritable, Text, BytesWritable> {
   
   	@Override
   	protected void reduce(Text key, Iterable<BytesWritable> values, Context context)		throws IOException, InterruptedException {
   
   		context.write(key, values.iterator().next());
   	}
   }
   ```

   

   （5）编写SequenceFileDriver类处理流程

   ```java
   package com.atguigu.mapreduce.inputformat;
   import java.io.IOException;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.BytesWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   import org.apache.hadoop.mapreduce.lib.output.SequenceFileOutputFormat;
   
   public class SequenceFileDriver {
   
   	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
   		
          // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
   		args = new String[] { "e:/input/inputinputformat", "e:/output1" };
   
          // 1 获取job对象
   		Configuration conf = new Configuration();
   		Job job = Job.getInstance(conf);
   
          // 2 设置jar包存储位置、关联自定义的mapper和reducer
   		job.setJarByClass(SequenceFileDriver.class);
   		job.setMapperClass(SequenceFileMapper.class);
   		job.setReducerClass(SequenceFileReducer.class);
   
          // 7设置输入的inputFormat
   		job.setInputFormatClass(WholeFileInputformat.class);
   
          // 8设置输出的outputFormat
   	 job.setOutputFormatClass(SequenceFileOutputFormat.class);
          
   // 3 设置map输出端的kv类型
   		job.setMapOutputKeyClass(Text.class);
   		job.setMapOutputValueClass(BytesWritable.class);
   		
          // 4 设置最终输出端的kv类型
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(BytesWritable.class);
   
          // 5 设置输入输出路径
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
          // 6 提交job
   		boolean result = job.waitForCompletion(true);
   		System.exit(result ? 0 : 1);
   	}
   }
   ```

   

## 3.2 MapReduce工作流程

1. 流程示意图,如下图所示

![img](MapReduce.assets/wps42.png)

![img](MapReduce.assets/wps43.png)

2. 流程详解

   上面的流程是整个MapReduce最全工作流程，但是Shuffle过程只是从第7步开始到第16步结束，具体Shuffle过程详解，如下：

   1）MapTask收集我们的map()方法输出的kv对，放到内存缓冲区中

   2）从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件

   3）多个溢出文件会被合并成大的溢出文件

   4）在溢出过程及合并的过程中，都要调用Partitioner进行分区和针对key进行排序

   5）ReduceTask根据自己的分区号，去各个MapTask机器上取相应的结果分区数据

   6）ReduceTask会取到同一个分区的来自不同MapTask的结果文件，ReduceTask会将这些文件再进行合并（归并排序）

   7）合并成大文件后，Shuffle的过程也就结束了，后面进入ReduceTask的逻辑运算过程（从文件中取出一个一个的键值对Group，调用用户自定义的reduce()方法）

3. 注意

   Shuffle中的缓冲区大小会影响到MapReduce程序的执行效率，原则上说，缓冲区越大，磁盘io的次数越少，执行速度就越快。

   缓冲区的大小可以通过参数调整，参数：io.sort.mb默认100M。

4. 源码解析流程

   ```java
   context.write(k, NullWritable.get());
   output.write(key, value);
   collector.collect(key, value,partitioner.getPartition(key, value, partitions));
   HashPartitioner();
   collect()
   close()
   collect.flush()
   sortAndSpill()
   sort()  QuickSort
   mergeParts();
   ```

   

   ​	![img](MapReduce.assets/wps44.jpg)

   collector.close();

   

## 3.3 Shuffle机制

### 3.3.1 Shuffle机制

Map方法之后，Reduce方法之前的数据处理过程称之为Shuffle。如图4-14所示

![img](MapReduce.assets/wps45.png)



### 3.3.2 Partition分区

![img](MapReduce.assets/wps46.png)

![img](MapReduce.assets/wps47.png)

![img](MapReduce.assets/wps48.png)

### 3.3.3 Partition分区案例实操

1. 需求

   将统计结果按照手机归属地不同省份输出到不同文件中（分区）

   （1）输入数据

   ​	![img](MapReduce.assets/wps49.png)

   （2）期望输出数据

   ​	手机号136、137、138、139开头都分别放到一个独立的4个文件中，其他开头的放到一个文件中。

2. 需求分析

   ![img](MapReduce.assets/wps50.png)

3. 在案例2.4的基础上，增加一个分区类

   ```java
   package com.atguigu.mapreduce.flowsum;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Partitioner;
   
   public class ProvincePartitioner extends Partitioner<Text, FlowBean> {
   
   	@Override
   	public int getPartition(Text key, FlowBean value, int numPartitions) {
   
   		// 1 获取电话号码的前三位
   		String preNum = key.toString().substring(0, 3);
   		
   		int partition = 4;
   		
   		// 2 判断是哪个省
   		if ("136".equals(preNum)) {
   			partition = 0;
   		}else if ("137".equals(preNum)) {
   			partition = 1;
   		}else if ("138".equals(preNum)) {
   			partition = 2;
   		}else if ("139".equals(preNum)) {
   			partition = 3;
   		}
   
   		return partition;
   	}
   }
   ```

   

4. 在驱动函数中增加自定义数据分区设置和ReduceTask设置

   ```java
   package com.atguigu.mapreduce.flowsum;
   import java.io.IOException;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class FlowsumDriver {
   
   	public static void main(String[] args) throws IllegalArgumentException, IOException, ClassNotFoundException, InterruptedException {
   
   		// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
   		args = new String[]{"e:/output1","e:/output2"};
   
   		// 1 获取配置信息，或者job对象实例
   		Configuration configuration = new Configuration();
   		Job job = Job.getInstance(configuration);
   
   		// 2 指定本程序的jar包所在的本地路径
   		job.setJarByClass(FlowsumDriver.class);
   
   		// 3 指定本业务job要使用的mapper/Reducer业务类
   		job.setMapperClass(FlowCountMapper.class);
   		job.setReducerClass(FlowCountReducer.class);
   
   		// 4 指定mapper输出数据的kv类型
   		job.setMapOutputKeyClass(Text.class);
   		job.setMapOutputValueClass(FlowBean.class);
   
   		// 5 指定最终输出的数据的kv类型
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(FlowBean.class);
   
   		// 8 指定自定义数据分区
   		job.setPartitionerClass(ProvincePartitioner.class);
   
   		// 9 同时指定相应数量的reduce task
   		job.setNumReduceTasks(5);
   		
   		// 6 指定job的输入原始文件所在目录
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
   		// 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
   		boolean result = job.waitForCompletion(true);
   		System.exit(result ? 0 : 1);
   	}
   }
   ```

   

### 3.3.4 WritableComparable排序

![img](MapReduce.assets/wps51.png)

![img](MapReduce.assets/wps52.png)

1. 排序的分类

![img](MapReduce.assets/wps53.png)

2. 自定义排序WritableComparable

   （1）原理分析

   bean对象做为key传输，需要实现WritableComparable接口重写compareTo方法，就可以实现排序。

   ```java
   @Override
   public int compareTo(FlowBean o) {
   
   	int result;
   		
   	// 按照总流量大小，倒序排列
   	if (sumFlow > bean.getSumFlow()) {
   		result = -1;
   	}else if (sumFlow < bean.getSumFlow()) {
   		result = 1;
   	}else {
   		result = 0;
   	}
   	return result;
   }
   ```

   

### 3.3.5 WritableComparable排序案例实操（全排序）

1. 需求

   根据案例2.3产生的结果再次对总流量进行排序。

   （1）输入数据

   原始数据             第一次处理后的数据

   ![img](MapReduce.assets/wps54.png)         ![img](MapReduce.assets/wps55.png)

   （2）期望输出数据

   13509468723	7335	110349	117684

   13736230513	2481	24681	27162

   13956435636	132		1512	1644

   13846544121	264		0		264

   。。。 。。。

2. 需求分析

   ![img](MapReduce.assets/wps56.png)

3. 代码实现

   （1）FlowBean对象在在需求1基础上增加了比较功能

   ```java
   package com.atguigu.mapreduce.sort;
   import java.io.DataInput;
   import java.io.DataOutput;
   import java.io.IOException;
   import org.apache.hadoop.io.WritableComparable;
   
   public class FlowBean implements WritableComparable<FlowBean> {
   
   	private long upFlow;
   	private long downFlow;
   	private long sumFlow;
   
   	// 反序列化时，需要反射调用空参构造函数，所以必须有
   	public FlowBean() {
   		super();
   	}
   
   	public FlowBean(long upFlow, long downFlow) {
   		super();
   		this.upFlow = upFlow;
   		this.downFlow = downFlow;
   		this.sumFlow = upFlow + downFlow;
   	}
   
   	public void set(long upFlow, long downFlow) {
   		this.upFlow = upFlow;
   		this.downFlow = downFlow;
   		this.sumFlow = upFlow + downFlow;
   	}
   
   	public long getSumFlow() {
   		return sumFlow;
   	}
   
   	public void setSumFlow(long sumFlow) {
   		this.sumFlow = sumFlow;
   	}	
   
   	public long getUpFlow() {
   		return upFlow;
   	}
   
   	public void setUpFlow(long upFlow) {
   		this.upFlow = upFlow;
   	}
   
   	public long getDownFlow() {
   		return downFlow;
   	}
   
   	public void setDownFlow(long downFlow) {
   		this.downFlow = downFlow;
   	}
   
   	/**
   	 * 序列化方法
   	 * @param out
   	 * @throws IOException
   	 */
   	@Override
   	public void write(DataOutput out) throws IOException {
   		out.writeLong(upFlow);
   		out.writeLong(downFlow);
   		out.writeLong(sumFlow);
   	}
   
   	/**
   	 * 反序列化方法 注意反序列化的顺序和序列化的顺序完全一致
   	 * @param in
   	 * @throws IOException
   	 */
   	@Override
   	public void readFields(DataInput in) throws IOException {
   		upFlow = in.readLong();
   		downFlow = in.readLong();
   		sumFlow = in.readLong();
   	}
   
   	@Override
   	public String toString() {
   		return upFlow + "\t" + downFlow + "\t" + sumFlow;
   	}
   
   	@Override
   	public int compareTo(FlowBean o) {
   		
   		int result;
   		
   		// 按照总流量大小，倒序排列
   		if (sumFlow > bean.getSumFlow()) {
   			result = -1;
   		}else if (sumFlow < bean.getSumFlow()) {
   			result = 1;
   		}else {
   			result = 0;
   		}
   
   		return result;
   	}
   }
   ```

   

   （2）编写Mapper类

   ```java
   package com.atguigu.mapreduce.sort;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class FlowCountSortMapper extends Mapper<LongWritable, Text, FlowBean, Text>{
   
   	FlowBean bean = new FlowBean();
   	Text v = new Text();
   
   	@Override
   	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
   
   		// 1 获取一行
   		String line = value.toString();
   		
   		// 2 截取
   		String[] fields = line.split("\t");
   		
   		// 3 封装对象
   		String phoneNbr = fields[0];
   		long upFlow = Long.parseLong(fields[1]);
   		long downFlow = Long.parseLong(fields[2]);
   		
   		bean.set(upFlow, downFlow);
   		v.set(phoneNbr);
   		
   		// 4 输出
   		context.write(bean, v);
   	}
   }
   ```

   

   （3）编写Reducer类

   ```java
   package com.atguigu.mapreduce.sort;
   import java.io.IOException;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class FlowCountSortReducer extends Reducer<FlowBean, Text, Text, FlowBean>{
   
   	@Override
   	protected void reduce(FlowBean key, Iterable<Text> values, Context context)	throws IOException, InterruptedException {
   		
   		// 循环输出，避免总流量相同情况
   		for (Text text : values) {
   			context.write(text, key);
   		}
   	}
   }
   ```

   

   （4）编写Driver类

   ```java
   package com.atguigu.mapreduce.sort;
   import java.io.IOException;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class FlowCountSortDriver {
   
   	public static void main(String[] args) throws ClassNotFoundException, IOException, InterruptedException {
   
   		// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
   		args = new String[]{"e:/output1","e:/output2"};
   
   		// 1 获取配置信息，或者job对象实例
   		Configuration configuration = new Configuration();
   		Job job = Job.getInstance(configuration);
   
   		// 2 指定本程序的jar包所在的本地路径
   		job.setJarByClass(FlowCountSortDriver.class);
   
   		// 3 指定本业务job要使用的mapper/Reducer业务类
   		job.setMapperClass(FlowCountSortMapper.class);
   		job.setReducerClass(FlowCountSortReducer.class);
   
   		// 4 指定mapper输出数据的kv类型
   		job.setMapOutputKeyClass(FlowBean.class);
   		job.setMapOutputValueClass(Text.class);
   
   		// 5 指定最终输出的数据的kv类型
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(FlowBean.class);
   
   		// 6 指定job的输入原始文件所在目录
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   		
   		// 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
   		boolean result = job.waitForCompletion(true);
   		System.exit(result ? 0 : 1);
   	}
   }
   ```

   

### 3.3.6 WritableComparable排序案例实操（区内排序）

1. 需求

   要求每个省份手机号输出的文件中按照总流量内部排序。

2. 需求分析

   基于前一个需求，增加自定义分区类，分区按照省份手机号设置

   ![img](MapReduce.assets/wps57.png)

3. 案例实操

   （1）增加自定义分区类

   ```java
   package com.atguigu.mapreduce.sort;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Partitioner;
   
   public class ProvincePartitioner extends Partitioner<FlowBean, Text> {
   
   	@Override
   	public int getPartition(FlowBean key, Text value, int numPartitions) {
   		
   		// 1 获取手机号码前三位
   		String preNum = value.toString().substring(0, 3);
   		
   		int partition = 4;
   		
   		// 2 根据手机号归属地设置分区
   		if ("136".equals(preNum)) {
   			partition = 0;
   		}else if ("137".equals(preNum)) {
   			partition = 1;
   		}else if ("138".equals(preNum)) {
   			partition = 2;
   		}else if ("139".equals(preNum)) {
   			partition = 3;
   		}
   
   		return partition;
   	}
   }
   ```

   

   （2）在驱动类中添加分区类

   ```java
   // 加载自定义分区类
   job.setPartitionerClass(ProvincePartitioner.class);
   
   // 设置Reducetask个数
   job.setNumReduceTasks(5);
   ```

   

### 3.3.7 Combiner合并

![img](MapReduce.assets/wps58.png)

（6）自定义Combiner实现步骤

​		（a）自定义一个Combiner继承Reducer，重写Reduce方法

```java
public class WordcountCombiner extends Reducer<Text, IntWritable, Text,IntWritable>{

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {

        // 1 汇总操作
		int count = 0;
		for(IntWritable v :values){
			count += v.get();
		}

        // 2 写出
		context.write(key, new IntWritable(count));
	}
}
```



​		（b）在Job驱动类中设置：

```java
job.setCombinerClass(WordcountCombiner.class);
```



### 3.3.8 Combiner合并案例实操

1. 需求

   统计过程中对每一个MapTask的输出进行局部汇总，以减小网络传输量即采用Combiner功能。

   （1）数据输入

   ![img](MapReduce.assets/wps59.png)

​	（2）期望输出数据

​	期望：Combine输入数据多，输出时经过合并，输出数据降低。

2. 需求分析

   ![img](MapReduce.assets/wps60.png)

3. 案例实操-方案一

   1）增加一个WordcountCombiner类继承Reducer

   ```java
   package com.atguigu.mr.combiner;
   import java.io.IOException;
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class WordcountCombiner extends Reducer<Text, IntWritable, Text, IntWritable>{
   
   IntWritable v = new IntWritable();
   
   	@Override
   	protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
   
           // 1 汇总
   		int sum = 0;
   
   		for(IntWritable value :values){
   			sum += value.get();
   		}
   
   		v.set(sum);
   
   		// 2 写出
   		context.write(key, v);
   	}
   }
   ```

   

   2）在WordcountDriver驱动类中指定Combiner

   ```java
   // 指定需要使用combiner，以及用哪个类作为combiner的逻辑
   job.setCombinerClass(WordcountCombiner.class);
   ```

   

4. 案例实操-方案二

   1）将WordcountReducer作为Combiner在WordcountDriver驱动类中指定

   ```java
   // 指定需要使用Combiner，以及用哪个类作为Combiner的逻辑
   job.setCombinerClass(WordcountReducer.class);
   ```

   运行程序，如图4-16，4-17所示

   ![img](MapReduce.assets/wps61.jpg) 

   图  未使用前

   ![img](MapReduce.assets/wps62.jpg) 

   图  使用后

### 3.3.9 GroupingComparator分组（辅助排序）

对Reduce阶段的数据根据某一个或几个字段进行分组。

分组排序步骤：

（1）自定义类继承WritableComparator

（2）重写compare()方法

```java
@Override
public int compare(WritableComparable a, WritableComparable b) {
		// 比较的业务逻辑
		return result;
}
```



（3）创建一个构造将比较对象的类传给父类

```java
protected OrderGroupingComparator() {
		super(OrderBean.class, true);
}
```



### 3.3.10 GroupingComparator分组案例实操

1. 需求

   有如下订单数据

​			                                                           表4-2 订单数据

| 订单id  | 商品id | 成交金额 |
| ------- | ------ | -------- |
| 0000001 | Pdt_01 | 222.8    |
| Pdt_02  | 33.8   |          |
| 0000002 | Pdt_03 | 522.8    |
| Pdt_04  | 122.4  |          |
| Pdt_05  | 722.4  |          |
| 0000003 | Pdt_06 | 232.8    |
| Pdt_02  | 33.8   |          |

​	现在需要求出每一个订单中最贵的商品。

​	（1）输入数据

​	![img](MapReduce.assets/wps63-1631538297867.png)

​	（2）期望输出数据

​	1	222.8

​	2	722.4

​	3	232.8

2. 需求分析

   （1）利用“订单id和成交金额”作为key，可以将Map阶段读取到的所有订单数据按照id升序排序，如果id相同再按照金额降序排序，发送到Reduce。

   （2）在Reduce端利用groupingComparator将订单id相同的kv聚合成组，然后取第一个即是该订单中最贵商品，如下图所示。

   ![img](MapReduce.assets/wps64.png)

3. 代码实现

   （1）定义订单信息OrderBean类

   ```java
   package com.atguigu.mapreduce.order;
   import java.io.DataInput;
   import java.io.DataOutput;
   import java.io.IOException;
   import org.apache.hadoop.io.WritableComparable;
   
   public class OrderBean implements WritableComparable<OrderBean> {
   
   	private int order_id; // 订单id号
   	private double price; // 价格
   
   	public OrderBean() {
   		super();
   	}
   
   	public OrderBean(int order_id, double price) {
   		super();
   		this.order_id = order_id;
   		this.price = price;
   	}
   
   	@Override
   	public void write(DataOutput out) throws IOException {
   		out.writeInt(order_id);
   		out.writeDouble(price);
   	}
   
   	@Override
   	public void readFields(DataInput in) throws IOException {
   		order_id = in.readInt();
   		price = in.readDouble();
   	}
   
   	@Override
   	public String toString() {
   		return order_id + "\t" + price;
   	}
   
   	public int getOrder_id() {
   		return order_id;
   	}
   
   	public void setOrder_id(int order_id) {
   		this.order_id = order_id;
   	}
   
   	public double getPrice() {
   		return price;
   	}
   
   	public void setPrice(double price) {
   		this.price = price;
   	}
   
   	// 二次排序
   	@Override
   	public int compareTo(OrderBean o) {
   
   		int result;
   
   		if (order_id > o.getOrder_id()) {
   			result = 1;
   		} else if (order_id < o.getOrder_id()) {
   			result = -1;
   		} else {
   			// 价格倒序排序
   			result = price > o.getPrice() ? -1 : 1;
   		}
   
   		return result;
   	}
   }
   ```

   （2）编写OrderSortMapper类

   ```java
   package com.atguigu.mapreduce.order;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class OrderMapper extends Mapper<LongWritable, Text, OrderBean, NullWritable> {
   
   	OrderBean k = new OrderBean();
   	
   	@Override
   	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
   		
   		// 1 获取一行
   		String line = value.toString();
   		
   		// 2 截取
   		String[] fields = line.split("\t");
   		
   		// 3 封装对象
   		k.setOrder_id(Integer.parseInt(fields[0]));
   		k.setPrice(Double.parseDouble(fields[2]));
   		
   		// 4 写出
   		context.write(k, NullWritable.get());
   	}
   }
   
   ```

   （3）编写OrderSortGroupingComparator类

   ```java
   
   package com.atguigu.mapreduce.order;
   import org.apache.hadoop.io.WritableComparable;
   import org.apache.hadoop.io.WritableComparator;
   
   public class OrderGroupingComparator extends WritableComparator {
   
   	protected OrderGroupingComparator() {
   		super(OrderBean.class, true);
   	}
   
   	@Override
   	public int compare(WritableComparable a, WritableComparable b) {
   
   		OrderBean aBean = (OrderBean) a;
   		OrderBean bBean = (OrderBean) b;
   
   		int result;
   		if (aBean.getOrder_id() > bBean.getOrder_id()) {
   			result = 1;
   		} else if (aBean.getOrder_id() < bBean.getOrder_id()) {
   			result = -1;
   		} else {
   			result = 0;
   		}
   
   		return result;
   	}
   }
   ```

   （4）编写OrderSortReducer类

   ```java
   
   package com.atguigu.mapreduce.order;
   import java.io.IOException;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class OrderReducer extends Reducer<OrderBean, NullWritable, OrderBean, NullWritable> {
   
   	@Override
   	protected void reduce(OrderBean key, Iterable<NullWritable> values, Context context)		throws IOException, InterruptedException {
   		
   		context.write(key, NullWritable.get());
   	}
   }
   ```

   （5）编写OrderSortDriver类

   ```java
   package com.atguigu.mapreduce.order;
   import java.io.IOException;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class OrderDriver {
   
   	public static void main(String[] args) throws Exception, IOException {
   
   // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
   		args  = new String[]{"e:/input/inputorder" , "e:/output1"};
   
   		// 1 获取配置信息
   		Configuration conf = new Configuration();
   		Job job = Job.getInstance(conf);
   
   		// 2 设置jar包加载路径
   		job.setJarByClass(OrderDriver.class);
   
   		// 3 加载map/reduce类
   		job.setMapperClass(OrderMapper.class);
   		job.setReducerClass(OrderReducer.class);
   
   		// 4 设置map输出数据key和value类型
   		job.setMapOutputKeyClass(OrderBean.class);
   		job.setMapOutputValueClass(NullWritable.class);
   
   		// 5 设置最终输出数据的key和value类型
   		job.setOutputKeyClass(OrderBean.class);
   		job.setOutputValueClass(NullWritable.class);
   
   		// 6 设置输入数据和输出数据路径
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
   		// 8 设置reduce端的分组
   	job.setGroupingComparatorClass(OrderGroupingComparator.class);
   
   		// 7 提交
   		boolean result = job.waitForCompletion(true);
   		System.exit(result ? 0 : 1);
   	}
   }
   ```

   



## 3.4 MapTask工作机制

MapTask工作机制如下图所示。

![img](MapReduce.assets/wps65.png)

​	（1）Read阶段：MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。

​	（2）Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。

​	（3）Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。

​	（4）Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。

​	溢写阶段详情：

​	步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。

​	步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。

​	步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。

​	（5）Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

​	当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。

​	在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。

​	让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。

## 3.5 ReduceTask工作机制

1. ReduceTask工作机制

   ReduceTask工作机制，如图4-19所示。

   ![img](MapReduce.assets/wps66.png)

   ​	（1）Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。

   ​	（2）Merge阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。

   ​	（3）Sort阶段：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。

   ​	（4）Reduce阶段：reduce()函数将计算结果写到HDFS上。

2. 设置ReduceTask并行度（个数）

   ReduceTask的并行度同样影响整个Job的执行并发度和执行效率，但与MapTask的并发数由切片数决定不同，ReduceTask数量的决定是可以直接手动设置：

   ```java
   // 默认值是1，手动设置为4
   job.setNumReduceTasks(4);
   ```

   

3. 实验：测试ReduceTask多少合适

   （1）实验环境：1个Master节点，16个Slave节点：CPU:8GHZ，内存: 2G

   （2）实验结论：

   ​							                         表    改变ReduceTask （数据量为1GB）

   | MapTask =16 |      |      |      |      |      |      |      |      |      |      |
   | ----------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
   | ReduceTask  | 1    | 5    | 10   | 15   | 16   | 20   | 25   | 30   | 45   | 60   |
   | 总时间      | 892  | 146  | 110  | 92   | 88   | 100  | 128  | 101  | 145  | 104  |

4. 注意事项

   ![img](file:///C:\Users\sjl\AppData\Local\Temp\ksohtml7808\wps67.png)



## 3.6 OutputFormat数据输出

### 3.6.1 OutputFormat接口实现类

![img](MapReduce.assets/wps68.png)

### 3.6.2 自定义OutputFormat

![img](MapReduce.assets/wps69.png)

### 3.6.3 自定义OoutputFormat案例实操

1. 需求

   ​	过滤输入的log日志，包含atguigu的网站输出到e:/atguigu.log，不包含atguigu的网站输出到e:/other.log。

   （1）输入数据

   ![img](MapReduce.assets/wps71.png)

   （2）期望输出数据

   ![img](MapReduce.assets/wps72.png) ![img](MapReduce.assets/wps73.png)

2. 需求分析

   ![img](MapReduce.assets/wps74.png)

3. 案例实操

   （1）编写FilterMapper类

   ```java
   package com.atguigu.mapreduce.outputformat;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class FilterMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
   	
   	@Override
   	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
   
   		// 写出
   		context.write(value, NullWritable.get());
   	}
   }
   ```

   （2）编写FilterReducer类

   ```java
   package com.atguigu.mapreduce.outputformat;
   import java.io.IOException;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class FilterReducer extends Reducer<Text, NullWritable, Text, NullWritable> {
   
   Text k = new Text();
   
   	@Override
   	protected void reduce(Text key, Iterable<NullWritable> values, Context context)		throws IOException, InterruptedException {
   
          // 1 获取一行
   		String line = key.toString();
   
          // 2 拼接
   		line = line + "\r\n";
   
          // 3 设置key
          k.set(line);
   
          // 4 输出
   		context.write(k, NullWritable.get());
   	}
   }
   ```

   （3）自定义一个OutputFormat类

   ```java
   package com.atguigu.mapreduce.outputformat;
   import java.io.IOException;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.RecordWriter;
   import org.apache.hadoop.mapreduce.TaskAttemptContext;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class FilterOutputFormat extends FileOutputFormat<Text, NullWritable>{
   
   	@Override
   	public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext job)			throws IOException, InterruptedException {
   
   		// 创建一个RecordWriter
   		return new FilterRecordWriter(job);
   	}
   }
   ```

   （4）编写RecordWriter类

   ```java
   package com.atguigu.mapreduce.outputformat;
   import java.io.IOException;
   import org.apache.hadoop.fs.FSDataOutputStream;
   import org.apache.hadoop.fs.FileSystem;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.RecordWriter;
   import org.apache.hadoop.mapreduce.TaskAttemptContext;
   
   public class FilterRecordWriter extends RecordWriter<Text, NullWritable> {
   
   	FSDataOutputStream atguiguOut = null;
   	FSDataOutputStream otherOut = null;
   
   	public FilterRecordWriter(TaskAttemptContext job) {
   
   		// 1 获取文件系统
   		FileSystem fs;
   
   		try {
   			fs = FileSystem.get(job.getConfiguration());
   
   			// 2 创建输出文件路径
   			Path atguiguPath = new Path("e:/atguigu.log");
   			Path otherPath = new Path("e:/other.log");
   
   			// 3 创建输出流
   			atguiguOut = fs.create(atguiguPath);
   			otherOut = fs.create(otherPath);
   		} catch (IOException e) {
   			e.printStackTrace();
   		}
   	}
   
   	@Override
   	public void write(Text key, NullWritable value) throws IOException, InterruptedException {
   
   		// 判断是否包含“atguigu”输出到不同文件
   		if (key.toString().contains("atguigu")) {
   			atguiguOut.write(key.toString().getBytes());
   		} else {
   			otherOut.write(key.toString().getBytes());
   		}
   	}
   
   	@Override
   	public void close(TaskAttemptContext context) throws IOException, InterruptedException {
   
   		// 关闭资源
   IOUtils.closeStream(atguiguOut);
   		IOUtils.closeStream(otherOut);	}
   }
   ```

   （5）编写FilterDriver类

   ```java
   package com.atguigu.mapreduce.outputformat;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class FilterDriver {
   
   	public static void main(String[] args) throws Exception {
   
   // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
   args = new String[] { "e:/input/inputoutputformat", "e:/output2" };
   
   		Configuration conf = new Configuration();
   		Job job = Job.getInstance(conf);
   
   		job.setJarByClass(FilterDriver.class);
   		job.setMapperClass(FilterMapper.class);
   		job.setReducerClass(FilterReducer.class);
   
   		job.setMapOutputKeyClass(Text.class);
   		job.setMapOutputValueClass(NullWritable.class);
   		
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(NullWritable.class);
   
   		// 要将自定义的输出格式组件设置到job中
   		job.setOutputFormatClass(FilterOutputFormat.class);
   
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   
   		// 虽然我们自定义了outputformat，但是因为我们的outputformat继承自fileoutputformat
   		// 而fileoutputformat要输出一个_SUCCESS文件，所以，在这还得指定一个输出目录
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
   		boolean result = job.waitForCompletion(true);
   		System.exit(result ? 0 : 1);
   	}
   }
   ```

   

## 3.7 Join多种应用

### 3.7.1 Reduce Join

![img](MapReduce.assets/wps75.png)



### 3.7.2 Reduce Join案例实操

1. 需求

   ![img](MapReduce.assets/wps76.png)

   表 4-4 订单数据表t_order

| id   | pid  | amount |
| ---- | ---- | ------ |
| 1001 | 01   | 1      |
| 1002 | 02   | 2      |
| 1003 | 03   | 3      |
| 1004 | 01   | 4      |
| 1005 | 02   | 5      |
| 1006 | 03   | 6      |

![img](MapReduce.assets/wps77.png)

 							表4-5 商品信息表t_product

| pid  | pname |
| ---- | ----- |
| 01   | 小米  |
| 02   | 华为  |
| 03   | 格力  |

​	将商品信息表中数据根据商品pid合并到订单数据表中。

​									表4-6 最终数据形式

| id   | pname | amount |
| ---- | ----- | ------ |
| 1001 | 小米  | 1      |
| 1004 | 小米  | 4      |
| 1002 | 华为  | 2      |
| 1005 | 华为  | 5      |
| 1003 | 格力  | 3      |
| 1006 | 格力  | 6      |

2. 需求分析

   通过将关联条件作为Map输出的key，将两表满足Join条件的数据并携带数据所来源的文件信息，发往同一个ReduceTask，在Reduce中进行数据的串联，如图4-20所示。

   ![img](MapReduce.assets/wps78.png)

   ​								图4-20 Reduce端表合并

3. 代码实现

   1）创建商品和订合并后的Bean类

```java
package com.atguigu.mapreduce.table;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.Writable;

public class TableBean implements Writable {

	private String order_id; // 订单id
	private String p_id;      // 产品id
	private int amount;       // 产品数量
	private String pname;     // 产品名称
	private String flag;      // 表的标记

	public TableBean() {
		super();
	}

	public TableBean(String order_id, String p_id, int amount, String pname, String flag) {

		super();

		this.order_id = order_id;
		this.p_id = p_id;
		this.amount = amount;
		this.pname = pname;
		this.flag = flag;
	}

	public String getFlag() {
		return flag;
	}

	public void setFlag(String flag) {
		this.flag = flag;
	}

	public String getOrder_id() {
		return order_id;
	}

	public void setOrder_id(String order_id) {
		this.order_id = order_id;
	}

	public String getP_id() {
		return p_id;
	}

	public void setP_id(String p_id) {
		this.p_id = p_id;
	}

	public int getAmount() {
		return amount;
	}

	public void setAmount(int amount) {
		this.amount = amount;
	}

	public String getPname() {
		return pname;
	}

	public void setPname(String pname) {
		this.pname = pname;
	}

	@Override
	public void write(DataOutput out) throws IOException {
		out.writeUTF(order_id);
		out.writeUTF(p_id);
		out.writeInt(amount);
		out.writeUTF(pname);
		out.writeUTF(flag);
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		this.order_id = in.readUTF();
		this.p_id = in.readUTF();
		this.amount = in.readInt();
		this.pname = in.readUTF();
		this.flag = in.readUTF();
	}

	@Override
	public String toString() {
		return order_id + "\t" + pname + "\t" + amount + "\t" ;
	}
}
```



​		2）编写TableMapper类

```java
package com.atguigu.mapreduce.table;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class TableMapper extends Mapper<LongWritable, Text, Text, TableBean>{

String name;
	TableBean bean = new TableBean();
	Text k = new Text();
	
	@Override
	protected void setup(Context context) throws IOException, InterruptedException {

		// 1 获取输入文件切片
		FileSplit split = (FileSplit) context.getInputSplit();

		// 2 获取输入文件名称
		name = split.getPath().getName();
	}

	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		
		// 1 获取输入数据
		String line = value.toString();
		
		// 2 不同文件分别处理
		if (name.startsWith("order")) {// 订单表处理

			// 2.1 切割
			String[] fields = line.split("\t");
			
			// 2.2 封装bean对象
			bean.setOrder_id(fields[0]);
			bean.setP_id(fields[1]);
			bean.setAmount(Integer.parseInt(fields[2]));
			bean.setPname("");
			bean.setFlag("order");
			
			k.set(fields[1]);
		}else {// 产品表处理

			// 2.3 切割
			String[] fields = line.split("\t");
			
			// 2.4 封装bean对象
			bean.setP_id(fields[0]);
			bean.setPname(fields[1]);
			bean.setFlag("pd");
			bean.setAmount(0);
			bean.setOrder_id("");
			
			k.set(fields[0]);
		}

		// 3 写出
		context.write(k, bean);
	}
}
```



​		3）编写TableReducer类

```java
package com.atguigu.mapreduce.table;
import java.io.IOException;
import java.util.ArrayList;
import org.apache.commons.beanutils.BeanUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class TableReducer extends Reducer<Text, TableBean, TableBean, NullWritable> {

	@Override
	protected void reduce(Text key, Iterable<TableBean> values, Context context)	throws IOException, InterruptedException {

		// 1准备存储订单的集合
		ArrayList<TableBean> orderBeans = new ArrayList<>();
		
// 2 准备bean对象
		TableBean pdBean = new TableBean();

		for (TableBean bean : values) {

			if ("order".equals(bean.getFlag())) {// 订单表

				// 拷贝传递过来的每条订单数据到集合中
				TableBean orderBean = new TableBean();

				try {
					BeanUtils.copyProperties(orderBean, bean);
				} catch (Exception e) {
					e.printStackTrace();
				}

				orderBeans.add(orderBean);
			} else {// 产品表

				try {
					// 拷贝传递过来的产品表到内存中
					BeanUtils.copyProperties(pdBean, bean);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}

		// 3 表的拼接
		for(TableBean bean:orderBeans){

			bean.setPname (pdBean.getPname());
			
			// 4 数据写出去
			context.write(bean, NullWritable.get());
		}
	}
}
```



​		4）编写TableDriver类

```java
package com.atguigu.mapreduce.table;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class TableDriver {

	public static void main(String[] args) throws Exception {
		
// 0 根据自己电脑路径重新配置
args = new String[]{"e:/input/inputtable","e:/output1"};

// 1 获取配置信息，或者job对象实例
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);

		// 2 指定本程序的jar包所在的本地路径
		job.setJarByClass(TableDriver.class);

		// 3 指定本业务job要使用的Mapper/Reducer业务类
		job.setMapperClass(TableMapper.class);
		job.setReducerClass(TableReducer.class);

		// 4 指定Mapper输出数据的kv类型
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(TableBean.class);

		// 5 指定最终输出的数据的kv类型
		job.setOutputKeyClass(TableBean.class);
		job.setOutputValueClass(NullWritable.class);

		// 6 指定job的输入原始文件所在目录
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
```



4. 测试

   运行程序查看结果

   ```csv
   1001	小米	1	
   1001	小米	1	
   1002	华为	2	
   1002	华为	2	
   1003	格力	3	
   1003	格力	3	
   ```

   ​	

5. 总结

   ![img](MapReduce.assets/wps79.png)

### 3.7.3 Map Join

1. 使用场景

   Map Join适用于一张表十分小、一张表很大的场景。

2. 优点

   思考：在Reduce端处理过多的表，非常容易产生数据倾斜。怎么办？

   在Map端缓存多张表，提前处理业务逻辑，这样增加Map端业务，减少Reduce端数据的压力，尽可能的减少数据倾斜。

3. 具体办法：采用DistributedCache

​	（1）在Mapper的setup阶段，将文件读取到缓存集合中。

​	（2）在驱动函数中加载缓存。

```java
// 缓存普通文件到Task运行节点。
job.addCacheFile(new URI("file://e:/cache/pd.txt"));
```



### 3.7.4 Map Join案例实操

1. 需求

![img](MapReduce.assets/wps80.png)

​													表4-4 订单数据表t_order

| id   | pid  | amount |
| ---- | ---- | ------ |
| 1001 | 01   | 1      |
| 1002 | 02   | 2      |
| 1003 | 03   | 3      |
| 1004 | 01   | 4      |
| 1005 | 02   | 5      |
| 1006 | 03   | 6      |

​													表4-5 商品信息表t_product

| pid  | pname |
| ---- | ----- |
| 01   | 小米  |
| 02   | 华为  |
| 03   | 格力  |

![img](MapReduce.assets/wps81.png)	

将商品信息表中数据根据商品pid合并到订单数据表中。

​															表4-6 最终数据形式

| id   | pname | amount |
| ---- | ----- | ------ |
| 1001 | 小米  | 1      |
| 1004 | 小米  | 4      |
| 1002 | 华为  | 2      |
| 1005 | 华为  | 5      |
| 1003 | 格力  | 3      |
| 1006 | 格力  | 6      |

2. 需求分析

   MapJoin适用于关联表中有小表的情形。

   ![img](MapReduce.assets/wps82.png)

​											图4-21 Map端表合并

3. 实现代码

   （1）先在驱动模块中添加缓存文件

   ```java
   package test;
   import java.net.URI;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class DistributedCacheDriver {
   
   	public static void main(String[] args) throws Exception {
   		
   // 0 根据自己电脑路径重新配置
   args = new String[]{"e:/input/inputtable2", "e:/output1"};
   
   // 1 获取job信息
   		Configuration configuration = new Configuration();
   		Job job = Job.getInstance(configuration);
   
   		// 2 设置加载jar包路径
   		job.setJarByClass(DistributedCacheDriver.class);
   
   		// 3 关联map
   		job.setMapperClass(DistributedCacheMapper.class);
   		
   // 4 设置最终输出数据类型
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(NullWritable.class);
   
   		// 5 设置输入输出路径
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
   		// 6 加载缓存数据
   		job.addCacheFile(new URI("file:///e:/input/inputcache/pd.txt"));
   		
   		// 7 Map端Join的逻辑不需要Reduce阶段，设置reduceTask数量为0
   		job.setNumReduceTasks(0);
   
   		// 8 提交
   		boolean result = job.waitForCompletion(true);
   		System.exit(result ? 0 : 1);
   	}
   }
   ```

   （2）读取缓存的文件数据

   ```java
   package test;
   import java.io.BufferedReader;
   import java.io.FileInputStream;
   import java.io.IOException;
   import java.io.InputStreamReader;
   import java.util.HashMap;
   import java.util.Map;
   import org.apache.commons.lang.StringUtils;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class DistributedCacheMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
   
   	Map<String, String> pdMap = new HashMap<>();
   	
   	@Override
   	protected void setup(Mapper<LongWritable, Text, Text, NullWritable>.Context context) throws IOException, InterruptedException {
   
   		// 1 获取缓存的文件
   		URI[] cacheFiles = context.getCacheFiles();
   		String path = cacheFiles[0].getPath().toString();
   		
   		BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream(path), "UTF-8"));
   		
   		String line;
   		while(StringUtils.isNotEmpty(line = reader.readLine())){
   
   			// 2 切割
   			String[] fields = line.split("\t");
   			
   			// 3 缓存数据到集合
   			pdMap.put(fields[0], fields[1]);
   		}
   		
   		// 4 关流
   		reader.close();
   	}
   	
   	Text k = new Text();
   	
   	@Override
   	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
   
   		// 1 获取一行
   		String line = value.toString();
   		
   		// 2 截取
   		String[] fields = line.split("\t");
   		
   		// 3 获取产品id
   		String pId = fields[1];
   		
   		// 4 获取商品名称
   		String pdName = pdMap.get(pId);
   		
   		// 5 拼接
   		k.set(line + "\t"+ pdName);
   		
   		// 6 写出
   		context.write(k, NullWritable.get());
   	}
   }
   ```

   

## 3.8 计数器应用

![img](MapReduce.assets/wps83.png)

## 3.9 数据清洗（ETL）

在运行核心业务MapReduce程序之前，往往要先对数据进行清洗，清理掉不符合用户要求的数据。清理的过程往往只需要运行Mapper程序，不需要运行Reduce程序。

### 3.9.1 数据清洗案例实操-简单解析版

1. 需求

   去除日志中字段长度小于等于11的日志。

   （1）输入数据

   ![img](MapReduce.assets/wps84.png)

   （2）期望输出数据

   每行字段长度都大于11。

2. 需求分析

   需要在Map阶段对输入的数据根据规则进行过滤清洗。

3. 实现代码

   （1）编写LogMapper类

   ```java
   package com.atguigu.mapreduce.weblog;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class LogMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
   	
   	Text k = new Text();
   	
   	@Override
   	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
   		
   		// 1 获取1行数据
   		String line = value.toString();
   		
   		// 2 解析日志
   		boolean result = parseLog(line,context);
   		
   		// 3 日志不合法退出
   		if (!result) {
   			return;
   		}
   		
   		// 4 设置key
   		k.set(line);
   		
   		// 5 写出数据
   		context.write(k, NullWritable.get());
   	}
   
   	// 2 解析日志
   	private boolean parseLog(String line, Context context) {
   
   		// 1 截取
   		String[] fields = line.split(" ");
   		
   		// 2 日志长度大于11的为合法
   		if (fields.length > 11) {
   
   			// 系统计数器
   			context.getCounter("map", "true").increment(1);
   			return true;
   		}else {
   			context.getCounter("map", "false").increment(1);
   			return false;
   		}
   	}
   }
   ```

   （2）编写LogDriver类

   ```java
   package com.atguigu.mapreduce.weblog;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class LogDriver {
   
   	public static void main(String[] args) throws Exception {
   
   // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
           args = new String[] { "e:/input/inputlog", "e:/output1" };
   
   		// 1 获取job信息
   		Configuration conf = new Configuration();
   		Job job = Job.getInstance(conf);
   
   		// 2 加载jar包
   		job.setJarByClass(LogDriver.class);
   
   		// 3 关联map
   		job.setMapperClass(LogMapper.class);
   
   		// 4 设置最终输出类型
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(NullWritable.class);
   
   		// 设置reducetask个数为0
   		job.setNumReduceTasks(0);
   
   		// 5 设置输入和输出路径
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
   		// 6 提交
   		job.waitForCompletion(true);
   	}
   }
   ```

   

### 3.9.2 数据清洗案例实操-复杂解析版

1. 需求

   对Web访问日志中的各字段识别切分，去除日志中不合法的记录。根据清洗规则，输出过滤后的数据。

   （1）输入数据

   ![img](MapReduce.assets/wps85.png)

   （2）期望输出数据

   都是合法的数据

2. 实现代码

   （1）定义一个bean，用来记录日志数据中的各数据字段

   ```java
   package com.atguigu.mapreduce.log;
   
   public class LogBean {
   	private String remote_addr;// 记录客户端的ip地址
   	private String remote_user;// 记录客户端用户名称,忽略属性"-"
   	private String time_local;// 记录访问时间与时区
   	private String request;// 记录请求的url与http协议
   	private String status;// 记录请求状态；成功是200
   	private String body_bytes_sent;// 记录发送给客户端文件主体内容大小
   	private String http_referer;// 用来记录从那个页面链接访问过来的
   	private String http_user_agent;// 记录客户浏览器的相关信息
   
   	private boolean valid = true;// 判断数据是否合法
   
   	public String getRemote_addr() {
   		return remote_addr;
   	}
   
   	public void setRemote_addr(String remote_addr) {
   		this.remote_addr = remote_addr;
   	}
   
   	public String getRemote_user() {
   		return remote_user;
   	}
   
   	public void setRemote_user(String remote_user) {
   		this.remote_user = remote_user;
   	}
   
   	public String getTime_local() {
   		return time_local;
   	}
   
   	public void setTime_local(String time_local) {
   		this.time_local = time_local;
   	}
   
   	public String getRequest() {
   		return request;
   	}
   
   	public void setRequest(String request) {
   		this.request = request;
   	}
   
   	public String getStatus() {
   		return status;
   	}
   
   	public void setStatus(String status) {
   		this.status = status;
   	}
   
   	public String getBody_bytes_sent() {
   		return body_bytes_sent;
   	}
   
   	public void setBody_bytes_sent(String body_bytes_sent) {
   		this.body_bytes_sent = body_bytes_sent;
   	}
   
   	public String getHttp_referer() {
   		return http_referer;
   	}
   
   	public void setHttp_referer(String http_referer) {
   		this.http_referer = http_referer;
   	}
   
   	public String getHttp_user_agent() {
   		return http_user_agent;
   	}
   
   	public void setHttp_user_agent(String http_user_agent) {
   		this.http_user_agent = http_user_agent;
   	}
   
   	public boolean isValid() {
   		return valid;
   	}
   
   	public void setValid(boolean valid) {
   		this.valid = valid;
   	}
   
   	@Override
   	public String toString() {
   
   		StringBuilder sb = new StringBuilder();
   		sb.append(this.valid);
   		sb.append("\001").append(this.remote_addr);
   		sb.append("\001").append(this.remote_user);
   		sb.append("\001").append(this.time_local);
   		sb.append("\001").append(this.request);
   		sb.append("\001").append(this.status);
   		sb.append("\001").append(this.body_bytes_sent);
   		sb.append("\001").append(this.http_referer);
   		sb.append("\001").append(this.http_user_agent);
   		
   		return sb.toString();
   	}
   }
   ```

   （2）编写LogMapper类

   ```java
   package com.atguigu.mapreduce.log;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class LogMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
   	Text k = new Text();
   	
   	@Override
   	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
   
   		// 1 获取1行
   		String line = value.toString();
   		
   		// 2 解析日志是否合法
   		LogBean bean = parseLog(line);
   		
   		if (!bean.isValid()) {
   			return;
   		}
   		
   		k.set(bean.toString());
   		
   		// 3 输出
   		context.write(k, NullWritable.get());
   	}
   
   	// 解析日志
   	private LogBean parseLog(String line) {
   
   		LogBean logBean = new LogBean();
   		
   		// 1 截取
   		String[] fields = line.split(" ");
   		
   		if (fields.length > 11) {
   
   			// 2封装数据
   			logBean.setRemote_addr(fields[0]);
   			logBean.setRemote_user(fields[1]);
   			logBean.setTime_local(fields[3].substring(1));
   			logBean.setRequest(fields[6]);
   			logBean.setStatus(fields[8]);
   			logBean.setBody_bytes_sent(fields[9]);
   			logBean.setHttp_referer(fields[10]);
   			
   			if (fields.length > 12) {
   				logBean.setHttp_user_agent(fields[11] + " "+ fields[12]);
   			}else {
   				logBean.setHttp_user_agent(fields[11]);
   			}
   			
   			// 大于400，HTTP错误
   			if (Integer.parseInt(logBean.getStatus()) >= 400) {
   				logBean.setValid(false);
   			}
   		}else {
   			logBean.setValid(false);
   		}
   		
   		return logBean;
   	}
   }
   ```

   （3）编写LogDriver类

   ```java
   package com.atguigu.mapreduce.log;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.NullWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class LogDriver {
   	public static void main(String[] args) throws Exception {
   		
   // 1 获取job信息
   		Configuration conf = new Configuration();
   		Job job = Job.getInstance(conf);
   
   		// 2 加载jar包
   		job.setJarByClass(LogDriver.class);
   
   		// 3 关联map
   		job.setMapperClass(LogMapper.class);
   
   		// 4 设置最终输出类型
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(NullWritable.class);
   
   		// 5 设置输入和输出路径
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
   		// 6 提交
   		job.waitForCompletion(true);
   	}
   }
   ```

   

## 3.10 MapReduce开发总结

在编写MapReduce程序时，需要考虑如下几个方面：

![img](MapReduce.assets/wps86.png)

![img](MapReduce.assets/wps87.png)

![img](MapReduce.assets/wps88.png)

![img](MapReduce.assets/wps89.png)

![img](MapReduce.assets/wps90.png)



# 4. Hadoop数据压缩

## 4.1 概述

![img](MapReduce.assets/wps91.png)

## 4.2 MR支持的压缩编码

![img](MapReduce.assets/wps92.png)

​															表4-7

| 压缩格式 | hadoop自带？ | 算法    | 文件扩展名 | 是否可切分 | 换成压缩格式后，原来的程序是否需要修改 |
| -------- | ------------ | ------- | ---------- | ---------- | -------------------------------------- |
| DEFLATE  | 是，直接使用 | DEFLATE | .deflate   | 否         | 和文本处理一样，不需要修改             |
| Gzip     | 是，直接使用 | DEFLATE | .gz        | 否         | 和文本处理一样，不需要修改             |
| bzip2    | 是，直接使用 | bzip2   | .bz2       | 是         | 和文本处理一样，不需要修改             |
| LZO      | 否，需要安装 | LZO     | .lzo       | 是         | 需要建索引，还需要指定输入格式         |
| Snappy   | 否，需要安装 | Snappy  | .snappy    | 否         | 和文本处理一样，不需要修改             |

为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器，如下表所示。

​																表4-8

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

压缩性能的比较

​																表4-9

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

http://google.github.io/snappy/

On a single core of a Core i7 processor in 64-bit mode, Snappy compresses at about 250 MB/sec or more and decompresses at about 500 MB/sec or more.

## 4.3 压缩方式选择

### 4.3.1 Gzip压缩

![img](MapReduce.assets/wps93.png)

### 4.3.2 Bzip2压缩

![img](MapReduce.assets/wps94.png)

### 4.3.3 Lzo压缩

![img](MapReduce.assets/wps95.png)

### 4.3.4 Snappy压缩

![img](MapReduce.assets/wps96.png)

## 4.4 压缩位置选择

压缩可以在MapReduce作用的任意阶段启用，如下图所示。

![img](MapReduce.assets/wps97.png)

![img](MapReduce.assets/wps98.png)



## 4.5 压缩参数配置

要在Hadoop中启用压缩，可以配置如下参数：

​								表4-10 配置参数

| 参数                                                         | 默认值                                                       | 阶段        | 建议                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ----------- | --------------------------------------------- |
| io.compression.codecs  （在core-site.xml中配置）             | org.apache.hadoop.io.compress.DefaultCodec, org.apache.hadoop.io.compress.GzipCodec, org.apache.hadoop.io.compress.BZip2Codec | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器  |
| mapreduce.map.output.compress（在mapred-site.xml中配置）     | false                                                        | mapper输出  | 这个参数设为true启用压缩                      |
| mapreduce.map.output.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec                   | mapper输出  | 企业多使用LZO或Snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress（在mapred-site.xml中配置） | false                                                        | reducer输出 | 这个参数设为true启用压缩                      |
| mapreduce.output.fileoutputformat.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress. DefaultCodec                  | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2       |
| mapreduce.output.fileoutputformat.compress.type（在mapred-site.xml中配置） | RECORD                                                       | reducer输出 | SequenceFile输出使用的压缩类型：NONE和BLOCK   |

## 4.6 压缩实操案例

### 4.6.1 数据流的压缩和解压缩

![img](MapReduce.assets/wps99.png)

测试一下如下压缩方式：

表4-11

| DEFLATE | org.apache.hadoop.io.compress.DefaultCodec |
| ------- | ------------------------------------------ |
| gzip    | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2   | org.apache.hadoop.io.compress.BZip2Codec   |

```java
package com.atguigu.mapreduce.compress;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.compress.CompressionCodec;
import org.apache.hadoop.io.compress.CompressionCodecFactory;
import org.apache.hadoop.io.compress.CompressionInputStream;
import org.apache.hadoop.io.compress.CompressionOutputStream;
import org.apache.hadoop.util.ReflectionUtils;

public class TestCompress {

	public static void main(String[] args) throws Exception {
		compress("e:/hello.txt","org.apache.hadoop.io.compress.BZip2Codec");
//		decompress("e:/hello.txt.bz2");
	}

	// 1、压缩
	private static void compress(String filename, String method) throws Exception {
		
		// （1）获取输入流
		FileInputStream fis = new FileInputStream(new File(filename));
		
		Class codecClass = Class.forName(method);
		
		CompressionCodec codec = (CompressionCodec) ReflectionUtils.newInstance(codecClass, new Configuration());
		
		// （2）获取输出流
		FileOutputStream fos = new FileOutputStream(new File(filename + codec.getDefaultExtension()));
		CompressionOutputStream cos = codec.createOutputStream(fos);
		
		// （3）流的对拷
		IOUtils.copyBytes(fis, cos, 1024*1024*5, false);
		
		// （4）关闭资源
		cos.close();
		fos.close();
fis.close();
	}

	// 2、解压缩
	private static void decompress(String filename) throws FileNotFoundException, IOException {
		
		// （0）校验是否能解压缩
		CompressionCodecFactory factory = new CompressionCodecFactory(new Configuration());

		CompressionCodec codec = factory.getCodec(new Path(filename));
		
		if (codec == null) {
			System.out.println("cannot find codec for file " + filename);
			return;
		}
		
		// （1）获取输入流
		CompressionInputStream cis = codec.createInputStream(new FileInputStream(new File(filename)));
		
		// （2）获取输出流
		FileOutputStream fos = new FileOutputStream(new File(filename + ".decoded"));
		
		// （3）流的对拷
		IOUtils.copyBytes(cis, fos, 1024*1024*5, false);
		
		// （4）关闭资源
		cis.close();
		fos.close();
	}
}
```





### 4.6.2 Map输出端采用压缩

即使你的MapReduce的输入输出文件都是未压缩的文件，你仍然可以对Map任务的中间结果输出做压缩，因为它要写在硬盘并且通过网络传输到Reduce节点，对其压缩可以提高很多性能，这些工作只要设置两个属性即可，我们来看下代码怎么设置。

1. 给大家提供的Hadoop源码支持的压缩格式有：`BZip2Codec` 、`DefaultCodec`

```java
package com.atguigu.mapreduce.compress;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.compress.BZip2Codec;	
import org.apache.hadoop.io.compress.CompressionCodec;
import org.apache.hadoop.io.compress.GzipCodec;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCountDriver {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

		Configuration configuration = new Configuration();

		// 开启map端输出压缩
	configuration.setBoolean("mapreduce.map.output.compress", true);
		// 设置map端输出压缩方式
	configuration.setClass("mapreduce.map.output.compress.codec", BZip2Codec.class, CompressionCodec.class);

		Job job = Job.getInstance(configuration);

		job.setJarByClass(WordCountDriver.class);

		job.setMapperClass(WordCountMapper.class);
		job.setReducerClass(WordCountReducer.class);

		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);

		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);

		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		boolean result = job.waitForCompletion(true);

		System.exit(result ? 1 : 0);
	}
}
```

2. Mapper保持不变

   ```java
   package com.atguigu.mapreduce.compress;
   import java.io.IOException;
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
   
   Text k = new Text();
   	IntWritable v = new IntWritable(1);
   
   	@Override
   	protected void map(LongWritable key, Text value, Context context)throws IOException, InterruptedException {
   
   		// 1 获取一行
   		String line = value.toString();
   
   		// 2 切割
   		String[] words = line.split(" ");
   
   		// 3 循环写出
   		for(String word:words){
   k.set(word);
   			context.write(k, v);
   		}
   	}
   }
   ```

3. Reducer保持不变

   ```java
   package com.atguigu.mapreduce.compress;
   import java.io.IOException;
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
   
   	IntWritable v = new IntWritable();
   
   	@Override
   	protected void reduce(Text key, Iterable<IntWritable> values,
   			Context context) throws IOException, InterruptedException {
   		
   		int sum = 0;
   
   		// 1 汇总
   		for(IntWritable value:values){
   			sum += value.get();
   		}
   		
           v.set(sum);
   
           // 2 输出
   		context.write(key, v);
   	}
   }
   ```

   

### 4.6.3 Reduce输出端采用压缩

基于WordCount案例处理。

1. 修改驱动

   ```java
   package com.atguigu.mapreduce.compress;
   import java.io.IOException;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.io.compress.BZip2Codec;
   import org.apache.hadoop.io.compress.DefaultCodec;
   import org.apache.hadoop.io.compress.GzipCodec;
   import org.apache.hadoop.io.compress.Lz4Codec;
   import org.apache.hadoop.io.compress.SnappyCodec;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class WordCountDriver {
   
   	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
   		
   		Configuration configuration = new Configuration();
   		
   		Job job = Job.getInstance(configuration);
   		
   		job.setJarByClass(WordCountDriver.class);
   		
   		job.setMapperClass(WordCountMapper.class);
   		job.setReducerClass(WordCountReducer.class);
   		
   		job.setMapOutputKeyClass(Text.class);
   		job.setMapOutputValueClass(IntWritable.class);
   		
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(IntWritable.class);
   		
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   		
   		// 设置reduce端输出压缩开启
   		FileOutputFormat.setCompressOutput(job, true);
   		
   		// 设置压缩的方式
   	    FileOutputFormat.setOutputCompressorClass(job, BZip2Codec.class); 
   //	    FileOutputFormat.setOutputCompressorClass(job, GzipCodec.class); 
   //	    FileOutputFormat.setOutputCompressorClass(job, DefaultCodec.class); 
   	    
   		boolean result = job.waitForCompletion(true);
   		
   		System.exit(result?1:0);
   	}
   }
   ```

   

2. Mapper和Reducer保持不变（详见4.6.2）

   

# 5. Yarn资源调度器

`Yarn`是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的`操作系统平台`，而`MapReduce`等运算程序则相当于运行于`操作系统之上的应用程序`。

## 5.1 Yarn基本架构

YARN主要由`ResourceManager`、`NodeManager`、`ApplicationMaster`和`Container`等组件构成，如下图所示。

![img](MapReduce.assets/wps100.png)

​													图  Yarn基本架构



## 5.2 Yarn工作机制

1. Yarn运行机制，如下图所示。

   ![img](MapReduce.assets/wps101.png)

2. 工作机制详解

   ​	（1）MR程序提交到客户端所在的节点。

   ​	（2）YarnRunner向`ResourceManager`申请一个`Application`。

   ​	（3）RM将该应用程序的资源路径返回给YarnRunner。

   ​	（4）该程序将运行所需资源提交到HDFS上。

   ​	（5）程序资源提交完毕后，申请运行`mrAppMaster`。

   ​	（6）RM将用户的请求初始化成一个Task。

   ​	（7）其中一个`NodeManager`领取到Task任务。

   ​	（8）该`NodeManager`创建容器`Container`，并产生`MRAppmaster`。

   ​	（9）`Container`从HDFS上拷贝资源到本地。

   ​	（10）`MRAppmaster`向RM 申请运行MapTask资源。

   ​	（11）RM将运行MapTask任务分配给另外两个`NodeManager`，另两个`NodeManager`分别领取任务并创建容器。

   ​	（12）MR向两个接收到任务的`NodeManager`发送程序启动脚本，这两个`NodeManager`分别启动MapTask，MapTask对数据分区排序。

   （13）`MrAppMaster`等待所有`MapTask`运行完毕后，向RM申请容器，运行`ReduceTask`。

   ​	（14）`ReduceTask`向`MapTask`获取相应分区的数据。

   ​	（15）程序运行完毕后，MR会向RM申请注销自己。

## 5.3 作业提交全过程

1. 作业提交过程之YARN，如下图所示。

   ![img](MapReduce.assets/wps102.png)

   作业提交全过程详解

   （1）作业提交

   第1步：Client调用`job.waitForCompletion`方法，向整个集群提交MapReduce作业。

   第2步：Client向RM申请一个作业id。

   第3步：RM给Client返回该job资源的提交路径和作业id。

   第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。

   第5步：Client提交完资源后，向RM申请运行`MrAppMaster`。

   （2）作业初始化

   第6步：当RM收到Client的请求后，将该job添加到容量调度器中。

   第7步：某一个空闲的NM领取到该Job。

   第8步：该NM创建Container，并产生`MRAppmaster`。

   第9步：下载Client提交的资源到本地。

   （3）任务分配

   第10步：`MrAppMaster`向RM申请运行多个`MapTask`任务资源。

   第11步：RM将运行MapTask任务分配给另外两个`NodeManager，另两个`NodeManager`分别领取任务并创建容器。

   （4）任务运行

   第12步：MR向两个接收到任务的`NodeManager`发送程序启动脚本，这两个`NodeManager`分别启动`MapTask`，`MapTask`对数据分区排序。

   第13步：`MrAppMaster`等待所有`MapTask`运行完毕后，向RM申请容器，运行`ReduceTask`。

   第14步：`ReduceTask`向`MapTask`获取相应分区的数据。

   第15步：程序运行完毕后，MR Appmater会向RM申请注销自己。

   （5）进度和状态更新

   YARN中的任务将其进度和状态(包括`counter`)返回给应用管理器（`AppMaster`）, 客户端每秒(通过`mapreduce.client.progressmonitor.pollinterval`设置)向应用管理器（`AppMaster`）请求进度更新, 展示给用户。

   （6）作业完成

   除了向应用管理器（`AppMaster`）请求作业进度外, 客户端每5秒都会通过调用`waitForCompletion()`来检查作业是否完成。时间间隔可以通过`mapreduce.client.completion.pollinterval`来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被`作业历史服务器`存储以备之后用户核查。

2. 作业提交过程之`MapReduce`，如下图所示：

   ![img](MapReduce.assets/wps103.png)

   

## 5.4 资源调度器

目前，Hadoop作业调度器主要有三种：`FIFO`、`Capacity Scheduler`和`Fair Scheduler`。Hadoop2.7.2默认的资源调度器是`Capacity Scheduler`。

具体设置详见：`yarn-default.xml`文件

```xml
<property>
    <description>The class to use as the resource scheduler.</description>
    <name>yarn.resourcemanager.scheduler.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>
```

1. 先进先出调度器（`FIFO`），如图4-27所示

   ![img](MapReduce.assets/wps104.png)

2. 容量调度器（Capacity Scheduler），如图4-28所示

   ![img](MapReduce.assets/wps105.png)

3. 公平调度器（Fair Scheduler），如图4-29所示

   ![img](MapReduce.assets/wps106.png)

   

## 5.5 任务的推测执行

1. 作业完成时间取决于最慢的任务完成时间

   一个作业由若干个Map任务和Reduce任务构成。因硬件老化、软件Bug等，某些任务可能运行非常慢。

   思考：系统中有99%的Map任务都完成了，只有少数几个Map老是进度很慢，完不成，怎么办？

2. 推测执行机制

   发现拖后腿的任务，比如某个任务运行速度远慢于任务平均速度。为拖后腿任务启动一个备份任务，同时运行。谁先运行完，则采用谁的结果。

3. 执行推测任务的前提条件

   （1）每个Task只能有一个备份任务

   （2）当前Job已完成的Task必须不小于0.05（5%）

   （3）开启推测执行参数设置。mapred-site.xml文件中默认是打开的。

   ```xml
   <property>
    	<name>mapreduce.map.speculative</name>
    	<value>true</value>
    	<description>If true, then multiple instances of some map tasks may be executed in parallel.</description>
   </property>
    
   <property>
    	<name>mapreduce.reduce.speculative</name>
    	<value>true</value>
    	<description>If true, then multiple instances of some reduce tasks may be executed in parallel.</description>
   </property>
   ```

4. 不能启用推测执行机制情况

     （1）任务间存在严重的负载倾斜；

     （2）特殊任务，比如任务向数据库中写数据。

5. 算法原理，如下图所示

   ![img](MapReduce.assets/wps107.png)

   

# 6. Hadoop企业优化

## 6.1 MapReduce跑的慢的原因

![img](MapReduce.assets/wps108.png)

## 6.2 MapReduce优化方法

`MapReduce`优化方法主要从六个方面考虑：数据输入、Map阶段、Reduce阶段、IO传输、数据倾斜问题和常用的调优参数。

### 6.2.1 数据输入

![img](MapReduce.assets/wps109.png)

### 6.2.2 Map阶段

![img](MapReduce.assets/wps110.png)

### 6.2.3 Reduce阶段

![img](MapReduce.assets/wps111.png)

![img](MapReduce.assets/wps112.png)

### 6.2.4 I/O传输

![img](MapReduce.assets/wps113.png)

### 6.2.5 数据倾斜问题

![img](MapReduce.assets/wps114.png)

![img](MapReduce.assets/wps115.png)

### 6.2.6 常用的调优参数

1. 资源相关参数

​	（1）以下参数是在用户自己的MR应用程序中配置就可以生效（mapred-default.xml）

​																表4-12

| 配置参数                                      | 参数说明                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| mapreduce.map.memory.mb                       | 一个MapTask可使用的资源上限（单位:MB），默认为1024。如果MapTask实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.reduce.memory.mb                    | 一个ReduceTask可使用的资源上限（单位:MB），默认为1024。如果ReduceTask实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.map.cpu.vcores                      | 每个MapTask可使用的最多cpu core数目，默认值: 1               |
| mapreduce.reduce.cpu.vcores                   | 每个ReduceTask可使用的最多cpu core数目，默认值: 1            |
| mapreduce.reduce.shuffle.parallelcopies       | 每个Reduce去Map中取数据的并行数。默认值是5                   |
| mapreduce.reduce.shuffle.merge.percent        | Buffer中的数据达到多少比例开始写入磁盘。默认值0.66           |
| mapreduce.reduce.shuffle.input.buffer.percent | Buffer大小占Reduce可用内存的比例。默认值0.7                  |
| mapreduce.reduce.input.buffer.percent         | 指定多少比例的内存用来存放Buffer中的数据，默认值是0.0        |

​	（2）应该在YARN启动之前就配置在服务器的配置文件中才能生效（yarn-default.xml）

​																	表4-13

| 配置参数                                 | 参数说明                                        |
| ---------------------------------------- | ----------------------------------------------- |
| yarn.scheduler.minimum-allocation-mb     | 给应用程序Container分配的最小内存，默认值：1024 |
| yarn.scheduler.maximum-allocation-mb     | 给应用程序Container分配的最大内存，默认值：8192 |
| yarn.scheduler.minimum-allocation-vcores | 每个Container申请的最小CPU核数，默认值：1       |
| yarn.scheduler.maximum-allocation-vcores | 每个Container申请的最大CPU核数，默认值：32      |
| yarn.nodemanager.resource.memory-mb      | 给Containers分配的最大物理内存，默认值：8192    |

（3）Shuffle性能优化的关键参数，应在YARN启动之前就配置好（mapred-default.xml）

表4-14

| 配置参数                         | 参数说明                          |
| -------------------------------- | --------------------------------- |
| mapreduce.task.io.sort.mb        | Shuffle的环形缓冲区大小，默认100m |
| mapreduce.map.sort.spill.percent | 环形缓冲区溢出的阈值，默认80%     |

2. 容错相关参数(MapReduce性能优化)

   ​																表4-15

| 配置参数                     | 参数说明                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| mapreduce.map.maxattempts    | 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.reduce.maxattempts | 每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.task.timeout       | Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个Task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该Task处于Block状态，可能是卡住了，也许永远会卡住，为了防止因为用户程序永远Block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是600000。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是“AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after 300 secsContainer killed by the ApplicationMaster.”。 |

## 6.3 HDFS小文件优化方法

### 6.3.1 HDFS小文件弊端

HDFS上每个文件都要在NameNode上建立一个索引，这个索引的大小约为150byte，这样当小文件比较多的时候，就会产生很多的索引文件，<font color=red>一方面会大量占用NameNode的内存空间，另一方面就是索引文件过大使得索引速度变慢。</font>

### 6.3.2 HDFS小文件解决方案

小文件的优化无非以下几种方式：

（1）在数据采集的时候，就将小文件或小批数据合成大文件再上传HDFS。

（2）在业务处理之前，在HDFS上使用MapReduce程序对小文件进行合并。

（3）在MapReduce处理时，可采用CombineTextInputFormat提高效率。

![img](MapReduce.assets/wps116.png)

![img](MapReduce.assets/wps117.png)

# 7. MapReduce扩展案例

## 7.1 倒排索引案例（多Job串联）

1. 需求

   有大量的文本（文档、网页），需要建立搜索索引，如图4-31所示。

   （1）数据输入

   ![img](MapReduce.assets/wps118.png) ![img](MapReduce.assets/wps119.png) ![img](MapReduce.assets/wps120.png)

   （2）期望输出数据

   atguigu	c.txt-->2	b.txt-->2	a.txt-->3	

   pingping	c.txt-->1	b.txt-->3	a.txt-->1	

   ss	c.txt-->1	b.txt-->1	a.txt-->2	

2. 需求分析

   ![img](MapReduce.assets/wps121.png)

3. 第一次处理

   （1）第一次处理，编写OneIndexMapper类

   ```java
   package com.atguigu.mapreduce.index;
   import java.io.IOException;
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   import org.apache.hadoop.mapreduce.lib.input.FileSplit;
   
   public class OneIndexMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
   	
   	String name;
   	Text k = new Text();
   	IntWritable v = new IntWritable();
   	
   	@Override
   	protected void setup(Context context)throws IOException, InterruptedException {
   
   		// 获取文件名称
   		FileSplit split = (FileSplit) context.getInputSplit();
   		
   		name = split.getPath().getName();
   	}
   	
   	@Override
   	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
   
   		// 1 获取1行
   		String line = value.toString();
   		
   		// 2 切割
   		String[] fields = line.split(" ");
   		
   		for (String word : fields) {
   
   			// 3 拼接
   			k.set(word+"--"+name);
   			v.set(1);
   			
   			// 4 写出
   			context.write(k, v);
   		}
   	}
   }
   ```

   

   （2）第一次处理，编写OneIndexReducer类

   ```java
   package com.atguigu.mapreduce.index;
   import java.io.IOException;
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   public class OneIndexReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
   	
   IntWritable v = new IntWritable();
   
   	@Override
   	protected void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {
   		
   		int sum = 0;
   
   		// 1 累加求和
   		for(IntWritable value: values){
   			sum +=value.get();
   		}
   		
          v.set(sum);
   
   		// 2 写出
   		context.write(key, v);
   	}
   }
   ```

   

   

   （3）第一次处理，编写OneIndexDriver类

   ```java
   package com.atguigu.mapreduce.index;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class OneIndexDriver {
   
   	public static void main(String[] args) throws Exception {
   
          // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
   		args = new String[] { "e:/input/inputoneindex", "e:/output5" };
   
   		Configuration conf = new Configuration();
   
   		Job job = Job.getInstance(conf);
   		job.setJarByClass(OneIndexDriver.class);
   
   		job.setMapperClass(OneIndexMapper.class);
   		job.setReducerClass(OneIndexReducer.class);
   
   		job.setMapOutputKeyClass(Text.class);
   		job.setMapOutputValueClass(IntWritable.class);
   		
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(IntWritable.class);
   
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
   		job.waitForCompletion(true);
   	}
   }
   ```

   

   （4）查看第一次输出结果

   ```java
   atguigu--a.txt	3
   atguigu--b.txt	2
   atguigu--c.txt	2
   pingping--a.txt	1
   pingping--b.txt	3
   pingping--c.txt	1
   ss--a.txt	2
   ss--b.txt	1
   ss--c.txt	1
   ```

   

4. 第二次处理

   （1）第二次处理，编写TwoIndexMapper类

   ```java
   package com.atguigu.mapreduce.index;
   import java.io.IOException;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   public class TwoIndexMapper extends Mapper<LongWritable, Text, Text, Text>{
   
   	Text k = new Text();
   	Text v = new Text();
   	
   	@Override
   	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
   		
   		// 1 获取1行数据
   		String line = value.toString();
   		
   		// 2用“--”切割
   		String[] fields = line.split("--");
   		
   		k.set(fields[0]);
   		v.set(fields[1]);
   		
   		// 3 输出数据
   		context.write(k, v);
   	}
   }
   ```

   

   （2）第二次处理，编写TwoIndexReducer类

   ```java
   package com.atguigu.mapreduce.index;
   import java.io.IOException;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   public class TwoIndexReducer extends Reducer<Text, Text, Text, Text> {
   
   Text v = new Text();
   
   	@Override
   	protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
   		// atguigu a.txt 3
   		// atguigu b.txt 2
   		// atguigu c.txt 2
   
   		// atguigu c.txt-->2 b.txt-->2 a.txt-->3
   
   		StringBuilder sb = new StringBuilder();
   
           // 1 拼接
   		for (Text value : values) {
   			sb.append(value.toString().replace("\t", "-->") + "\t");
   		}
   
   v.set(sb.toString());
   
   		// 2 写出
   		context.write(key, v);
   	}
   }
   ```

   

   （3）第二次处理，编写TwoIndexDriver类

   ```java
   package com.atguigu.mapreduce.index;
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Job;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   
   public class TwoIndexDriver {
   
   	public static void main(String[] args) throws Exception {
   
          // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
   args = new String[] { "e:/input/inputtwoindex", "e:/output6" };
   
   		Configuration config = new Configuration();
   		Job job = Job.getInstance(config);
   
   job.setJarByClass(TwoIndexDriver.class);
   		job.setMapperClass(TwoIndexMapper.class);
   		job.setReducerClass(TwoIndexReducer.class);
   
   		job.setMapOutputKeyClass(Text.class);
   		job.setMapOutputValueClass(Text.class);
   		
   		job.setOutputKeyClass(Text.class);
   		job.setOutputValueClass(Text.class);
   
   		FileInputFormat.setInputPaths(job, new Path(args[0]));
   		FileOutputFormat.setOutputPath(job, new Path(args[1]));
   
   		boolean result = job.waitForCompletion(true);
   System.exit(result?0:1);
   	}
   }
   ```

   

   （4）第二次查看最终结果

   ```java
   atguigu	c.txt-->2	b.txt-->2	a.txt-->3	
   pingping	c.txt-->1	b.txt-->3	a.txt-->1	
   ss	c.txt-->1	b.txt-->1	a.txt-->2	
   ```

   

## 7.2 TopN案例

TODO

## 7.3 找博客共同好友案例

TODO

# 8. 常见错误及解决方案

1. 导包容易出错。尤其Text和CombineTextInputFormat。

2. Mapper中第一个输入的参数必须是LongWritable或者NullWritable，不可以是IntWritable.  报的错误是类型转换异常。

3. java.lang.Exception: java.io.IOException: Illegal partition for 13926435656 (4)，说明Partition和ReduceTask个数没对上，调整ReduceTask个数。

4. 如果分区数不是1，但是reducetask为1，是否执行分区过程。答案是：不执行分区过程。因为在MapTask的源码中，执行分区的前提是先判断ReduceNum个数是否大于1。不大于1肯定不执行。

5. 在Windows环境编译的jar包导入到Linux环境中运行，

   ```shell
   hadoop jar wc.jar com.atguigu.mapreduce.wordcount.WordCountDriver /user/atguigu/ /user/atguigu/output
   
   报如下错误：
   
   Exception in thread "main" java.lang.UnsupportedClassVersionError: com/atguigu/mapreduce/wordcount/WordCountDriver : Unsupported major.minor version 52.0
   
   原因是Windows环境用的jdk1.7，Linux环境用的jdk1.8。
   ```

   解决方案：统一jdk版本。

6. 缓存pd.txt小文件案例中，报找不到pd.txt文件

   原因：大部分为路径书写错误。还有就是要检查pd.txt.txt的问题。还有个别电脑写相对路径找不到pd.txt，可以修改为绝对路径。

7. 报类型转换异常。

   通常都是在驱动函数中设置Map输出和最终输出时编写错误。

   Map输出的key如果没有排序，也会报类型转换异常。

8. 集群中运行wc.jar时出现了无法获得输入文件。

   原因：WordCount案例的输入文件不能放用HDFS集群的根目录。

9. 出现了如下相关异常

   ```shell
   Exception in thread "main" java.lang.UnsatisfiedLinkError: org.apache.hadoop.io.nativeio.NativeIO$Windows.access0(Ljava/lang/String;I)Z
   	at org.apache.hadoop.io.nativeio.NativeIO$Windows.access0(Native Method)
   	at org.apache.hadoop.io.nativeio.NativeIO$Windows.access(NativeIO.java:609)
   	at org.apache.hadoop.fs.FileUtil.canRead(FileUtil.java:977)
   
   java.io.IOException: Could not locate executable null\bin\winutils.exe in the Hadoop binaries.
   	at org.apache.hadoop.util.Shell.getQualifiedBinPath(Shell.java:356)
   	at org.apache.hadoop.util.Shell.getWinUtilsPath(Shell.java:371)
   	at org.apache.hadoop.util.Shell.<clinit>(Shell.java:364)
   ```

   解决方案：拷贝hadoop.dll文件到Windows目录C:\Windows\System32。个别同学电脑还需要修改Hadoop源码。

   方案二：创建如下包名，并将NativeIO.java拷贝到该包名下

   ![img](MapReduce.assets/wps122.jpg) ![img](MapReduce.assets/wps123.png)

10. 自定义Outputformat时，注意在RecordWirter中的close方法必须关闭流资源。否则输出的文件内容中数据为空。

```java
@Override
public void close(TaskAttemptContext context) throws IOException, InterruptedException {
		if (atguigufos != null) {
			atguigufos.close();
		}
		if (otherfos != null) {
			otherfos.close();
		}
}
```



