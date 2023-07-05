

# 1. 概述

学习框架

![image-20230621170902654](HBase2.x.assets/image-20230621170902654.png)

Hadoop与HBase兼容性：

![hadoop和HBase的兼容性](HBase2.x.assets/hadoop和HBase的兼容性.png)

## 1.1 HBase定义

`Apache HBase`是以HDFS为数据存储的，一种分布式、可扩展的**NoSQL数据库**。

HBase = Hadoop Database

HBase支持数十亿行，数百万列。

## 1.2 HBase的数据模型

HBase的设计理念依据Google的Big Table论文，论文中对于数据模型的首句介绍：

`BigTable`是一个**稀疏的**、**分布式的**、**持久的**多维排序map。

- **稀疏的**

  稀疏性是HBase的 一个突出特点。Hbase整整一行仅有一列有值，其他列都为空值。在其他数据库中，对于空值的处理一般会填充null，而Hbase的空值不需要任何填充。因为HBase的列在理论上是允许无限扩展的，对于成百万列的表来说，通常会存在大量的空值，如果使用填充null的策略，势必会浪费大量的空间，因此稀疏性是HBase的列可以无限扩展的一个重要条件。

- **分布式的**

  很容易理解，构成HBase的所有Map并不集中在某台机器上，而是分布在整个集群中。

- **持久的**

  数据是持久化存储的，服务器重启不会造成数据丢失。

- **多维的**

  Hbase的key是一个复合数据结构，由多维元素构成，包括rowkey、columnfamily、qualifier、type以及timestamp。

- **排序的**

  构成HBase的KV在同一个文件中都是有序的，但规则并不是仅仅按照rowkey排序，而是按照KV中的key进行排序————先比较rowkey，rowkey小的排在前面；如果rowkey相同，再比较column，即columnfamily：qualifier，colum小的排在前面；如果column还相同，再比较时间戳timestamp，即版本信息，timestamp大的排在前面。

- **map**

  map即键映射到值，就像RDBMS中的key-value一样。HBase中的映射是由行键（rowkey）、列族（columnfamily）、列（column）和时间戳（timestamp）组成键，映射中的每个值都是一个未解释的字节数组。

> HBase使用与BigTable非常相似的数据模型，用户将数据行存储在**带标签的**表中。数据行具有可排序的键和任意数量的列。该表存储稀疏，因此如果用户喜欢，同一表中的行可以具有疯狂变化的列。

### 1.2.1 HBase的逻辑存储结构

HBase 可以用于存储多种结构的数据，以 JSON 为例，存储的数据原貌为：

```json
{
  "row_key1":{
    "personal_info":{
    "name":"zhangsan",
    "city":"北京",
    "phone":"131********"
    },
		"office_info":{
      "tel":"010-1111111",
      "address":"atguigu"
      }
		},
  "row_key11":{
    "personal_info":{
      "city":"上海",
      "phone":"132********"
    },
  	"office_info":{
  		"tel":"010-1111111" 
  	}
  },
  "row_key2":{
  ......
}
```

上面的json对应的HBase的逻辑存储结构。

![image-20230621175415382](HBase2.x.assets/image-20230621175415382.png)

HBase存储的数据是稀疏的，数据存储多维，不同的行可以具有不同的列。

数据存储整体有序，按照RowKey的字典序排列，RowKey为Byte数组。

**StoreFile存储一个列族中的多行数据。**

**Region存储多个列族中的多行数据，所以Region由多个StoreFile组成。**

### 1.2.2 HBase的物理存储结构

物理存储结构即为数据映射关系，而在概念视图的空单元格，底层实际根本不存储。

![image-20230621180342982](HBase2.x.assets/image-20230621180342982.png)

由于HBase的底层数据存储为HDFS，而HDFS是不支持直接对文件进行修改的，那么HBase如何实现对某行的数据进行修改呢？

**答**：通过Timestamp实现不同版本的数据，读取的时候直接读取新版本的数据，就认为是对旧数据进行的修改。

HBase数据的底层存储文件为`StoreFile`。

### 1.2.3 数据模型

- NameSpace

  命令空间，**类似于关系型数据库中的database的概念**，每个命名空间下有多个表。HBase有两个自带的命名空间：`hbase`和`default`。`hbase`中存放的是HBase内置的表，`default`是用户默认使用的命名空间。

- Table

  类似于关系型数据库中的表的概念，不同的是，**HBase定义表时只需要声明列族即可**，不需要声明具体的列。因为数据存储是稀疏的，所有往HBase写入数据时，字段可以**动态**、**按需**制定。因此，和关系型数据库相比，HBase能够轻松应对字段变更的场景。

- Row

  HBase表中的每行数据都由一个**RowKey**和多个**Column**组成，数据是按照RowKey的字典序存储的，并且**查询数据时只能根据RowKey进行检索**，所以RowKer的设计十分重要。

- Column

  HBase中的每个列都由**Column Family(列族)**和**Column Qualifier(列限定符)**进行限定，例如 `info:name`, `info:age`。建表时，只需制定列族，而列限定符无需预先定义。

- Time Stamp

  用于表示数据的不同版本（version），每条数据写入时，系统会自动为其加上该字段，其值为写入HBase的时间。

- Cell

  由`{rowkey, column Family, column Qualifier, timestamp}` 唯一确定的单元。cell中的数据全部是字节码形式存储。即同样的rowkey，列族，列，不同的时间戳代表是两个不同的cell。

## 1.3 HBase的架构

### 1.3.1 HBase的基础架构

![image-20230625095719302](HBase2.x.assets/image-20230625095719302.png)

架构中的角色：

- **Master**

  实现类为`HMaster`，负责监控集群中所有的`RegionServer`实例，主要作用如下：

  - 管理元数据表格`hbase:meta`，接受用户对表格的创建/修改/删除的命令并执行
  - 监控region是否需要进行负载均衡，故障转移和region的拆分

  通过启动多个后台线程监控实现上述功能：

  - `LoadBalancer`负载均衡器

    周期性监控region分布在RegionServer上是否均衡，由参数`hbase.balancer.period`控制周期时间，默认5分钟。

  - `CatalogJanitor`元数据管理器

    定期检查和清理`hbase:meta`中的数据。meta表内容在进阶中介绍。

  - `MasterProcWAL` master预写日志处理器

    把master需要执行的任务记录到预写日志WAL中，如果master宕机，让`backupMaster`读取WAL日志继续干。

- **Region Server**

  Region Server实现类为`HRegionServer`，主要作用如下：

  - 负责数据cell的处理，例如写入数据put，查询数据get等
  - 拆分/合并region的实际执行者，由master监控，由regionServer执行。

- **ZooKeeper**

  HBase通过ZooKeeper来做master的高可用，记录RegionServer的部署信息，并存储有meta表的位置信息。

  **HBase对于数据的读写操作时直接访问ZK**，在2.3版本退出了`Master Registry`模式，客户端可以直接访问master，使用此功能，会加大对master的压力，减轻对ZK的压力

- **HDFS**

  HDFS为HBase提供底层数据存储，同时为HBase提供高容错的支持。

### 1.3.2 HBase的HA架构

# 2. 部署

## 2.1 部署ZooKeeper

需要保证ZK集群正常部署，并在每个ZK的节点启动ZK进程。

```bash
bin/zkServer.sh start
```

## 2.2 部署Hadoop

需要Hadoop集群的正常部署并启动。

```bash
sbin/start-dfs.sh
sbin/start-yarn.sh
```

## 2.3 部署HBase

### 2.3.1 下载

下载HBase

官网下载比较慢，这里选择华为云国内源进行下载。下载地址：https://repo.huaweicloud.com/apache/hbase

下载Phoenix：https://repo.huaweicloud.com/apache/phoenix

版本：

- `hbase-2.5.5-bin.tar.gz`
- `phoenix-hbase-2.5-5.1.3-bin.tar.gz`



### 2.3.2 解压

```bash
tar -zxvf hbase-2.5.5-bin.tar.gz -C /opt/modules/
```

解压phoenix

```bash
tar -zxvf phoenix-hbase-2.5-5.1.3-bin.tar.gz -C /opt/modules
```

### 2.3.3 配置HBase环境变量

修改HBASE环境变量

```bash
sudo vim /etc/profile.d/hadoop322_env.sh
# 增加
# HBASE_HOME
export HBASE_HOME=/opt/modules/hbase-2.5.5
export PATH=$PATH:$HBASE_HOME/bin
```

分发到所有节点

```bash
sudo /home/hadoop/bin/xsync -files /etc/profile -hostsFile ~/workers
```

source以让环境变量生效。xshell > 工具 > 发送键输入到所有会话。

```bash
source /etc/profile
```

### 2.3.4 配置

#### 2.3.4.1 **hbase-env.sh**

```bash
export HBASE_MANAGES_ZK=false 				# 表示不使用HBase内置的ZK。
```

#### 2.3.4.2 **hbase-site.xml**

```xml
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
    <description>hbase是否部署为分布式模式</description>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>hadoop322-node01,hadoop322-node02,hadoop322-node03</value>
    <description>zk集群的主机名</description>
  </property>
	<!--
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/modules/zookeeper/zkData</value>
    <description>zk中的数据存储在本地的什么目录下，因为在zk的配置文件中已经修改过了，所以这里注释掉。</description>
  </property>
	-->
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://mycluster/hbase</value>
    <description>hbase的数据存储在hdfs上的什么目录下,启动hbase时如果此目录不存在会自动创建的</description>
  </property>
</configuration>
```

#### 2.3.4.3 **regionservers**

```bash
hadoop322-node01
hadoop322-node02
hadoop322-node03
```

#### 2.3.4.4 HBase和Hadoop的log4j兼容性问题

Hadoop，HBase都会有自己的log4j包，可能就会存在兼容性问题，需要修改HBase的jar包，使用Hadoop的jar包。

TODO 需要补全如下命令

```bash
mv /opt/moduels/hbase-2.5.5/lib/client-facing-thirdparty
```

#### 2.3.4.5 将修改同步到其他hbase节点

```bash
xsync -file conf -hostsFile ~/workers
xsync -file lib -hostsFile ~/workers
```

### 2.3.5 启动HBase

#### 2.3.5.1 启动单个节点上的hbase进程

```bash
bin/hbase-daemon.sh start master
bin/hbase-daemon.sh start regionserver
```

#### 2.3.5.2 停止单个节点上的hbase进程

```bash
bin/hbase-daemon.sh stop master
bin/hbase-daemon.sh stop regionserver
```

#### 2.3.5.3 启动整个hbase集群的进程

```bash
bin/start-hbase.sh
```

#### 2.3.5.4 停止整个hbase集群的进程

```bash
bin/stop-hbase.sh
```

#### 2.3.5.5 查看HBase页面

启动成功后，可以通过`host:port`的方式查看HBase的管理页面，例如`http://hadoop322-node01:16010`

## 2.4 部署HBase高可用集群

在HBase中HMaster负责监控HRegionServer的生命周期，均衡RegionServer的负载，如果HMaster挂了，那么整个HBase集群将陷入不健康的状态，并且此时的工作状态不会维持太久。所以HBase支持对HMaster进行高可用配置。

**（1）关闭HBase集群（没有启动HBase集群则跳过）**

```bash
bin/stop-hbase.sh
```

**（2）在conf目录下创建backup-masters文件**

```bash
touch conf/backup-masters
```

**（3）在backup-masters问价中配置高可用HMaster节点**

```bash
echo hadoop322-node03 > conf/backup-masters
```

**（4）将整个conf目录同步到其他节点**

```bash
xsync -file conf -hostsFile ~/workers
```

**（5）重启hbase，验证**

```bash
bin/start-hbase.sh
```

浏览器打开：http://hadoop322-node01:16010

http://hadoop322-node03:16010

**（6）模拟Master挂掉**

```bash
# 1、kill 当前的master
kill -9 pid
# 2、节点进程查看，管理页面查看
jps
http://hadoop322-node01:16010
# 3、查看backup master的管理页面，发现以成为新的master
http://hadoop322-node03:16010
# 4、启动原master进程，现在会变成backup master进程
登录节点
bin/hbase-daemon.sh start master
```

# 3. 使用

## 3.1 HBase Shell

### 3.1.1 基本操作

```bash
hbase shell

> help "command"
```

### 3.1.2 namespace操作

```bash
alter_namespace
create_namespace 'ns1'
describe_namespace
drop_namespace
list_namespace
list_namespace 'abc.*'
list_namespace_tables
```

### 3.1.3 DLL操作

```bash
alter
alter_sync
alter_status
clone_table_schema
create
describe
disable
disable_all
drop
drop_all
enable
enable_all
exists
get_table
is_disabled
is_enabled
list
list_regions
locate_region
show_filters
```

**创建表**

```bash
help "create"
create 'ns1:t1', {NAME =>'cf1', VERSIONS=>5}, {NAME=>'cf2'}
create 'ns1:t1', 'cf1', 'cf2', 'cf3'
create 'ns1:student1', 'info'
```

VERSIONS=>5，表示该列族中的列保留多少个版本，同一个rowkey，同一个列族:列，再加时间戳就是一个版本。

**查看表**

```bash
list
describe 'ns1:t1'
```

**修改表**

表名创建时写的所有和列族相关的信息，都可以后续通过alter语句修改，包括增加删除列族。

（1）增加列族和修改信息都使用覆盖的方法

```bash
alter 'student1', {NAME=>'cf1', VERSIONS=>3}
```

（2）删除信息使用特殊语法。删除列族

```bash
alter 'student1', NAME=>'cf1', METHOD=>'delete'
alter 'student1', 'delete'=>'cf1'
```

**删除表**

`hbase shell`中删除表格，需要现将表格状态设置为不可用。

```bash
disable 'ns1:student1'
drop 'student1'
```



### 3.1.4 DML操作

```bash
append
count
delete
deleteall
get
get_counter
get_splits
incr
put
scan
truncate
truncate_preserve
```

**写入数据**

在HBase中如果想要写入数据，只能添加结构中最底层的cell。可以手动写入时间戳制定cell的版本，推荐不写默认使用当前**系统时间**。

```bash
# put 'biao', 'rowkey', 'cf:col', 'value'
put 'ns1:student', '1001', 'info:name', 'zhangsan'
put 'ns1:student', '1001', 'info:name', 'lisi'
put 'ns1:student', '1001', 'info:age', '18'
```

如果重复写入相同的rowkey，相同列的数据，会写入多个版本进行覆盖。是通过时间戳进行覆盖。

**读取数据**

读取的方法有两个：`get`和`scan`。

get最大范围是一行数据，也可以进行列的过滤，读取数据的结果为多行cell。

```bash
get 'ns1:student', '1001'
# get 1个列
get 'ns1:student', '1001', {COLUMNS=>'info:name'}
# get多个列
get 'ns1:student', '1001', {COLUMNS=>['info:name', 'info:age']}

# 读取制定rowkey多个版本的数据，表示最多读取6个版本的数据
get 'ns1:student', '1001', {COLUMN=>'info:name', VERSIONS=>6}
```

![image-20230626210940319](HBase2.x.assets/image-20230626210940319.png)

**scan**是扫描数据，能够读取多行数据，不建议扫描过多的数据。推荐使用STARTROW，STOPROW来控制读取的数据。默认范围**左闭右开**。

```bash
scan 'ns1:student', {STARTROW=>'1001', STOPROW=>'1003'}
```

<font color="red">实际开发中使用 shell 的机会不多，所有丰富的使用方法到 API 中介绍。</font>

**删除数据**

删除数据的方法有两个：`delete`和`deleteall`

==delete表示删除一个版本的数据，即为1个cell，不填写版本默认删除最新的一个版本。==

如果有多个版本，那么删除最新版之后，再次查看数据为之前的次新版了。

```bash
# 删除最新版本
delete 'ns1:student', '1001', 'info:name'
# 删除指定时间戳之前的一个版本的数据
delete 'ns1:student', '1001', 'info:name', 时间戳
```

==deleteall表示删除所有版本的数据，即为当前行当前列的多个cell。==<font color="red">（执行命令会标记数据为要删除，不会直接将数据彻底删除，删除数据只在特定时期清理磁盘时进行）</font>

```bash
deleteall 'ns1:student', '1001', 'info:name'
```





### 3.2.1 环境准备

TODO

### 3.2.2 创建连接

### 3.2.3 DDL操作

### 3.2.4 DML操作



# 4. 配置

# 5. 原理

## 5.1 Master架构

![image-20230626215229424](HBase2.x.assets/image-20230626215229424.png)

Meta表介绍：（<font color="red">警告：不要去改这个表</font>）

全称`hbase:meta`，只是在list命令中被过滤了，本质上和HBase的其他表一样。

- `RowKey`

  `([table],[region start key],[region id])` 即 表名，region 起始位置和 regionID。

- 列：

  - `info:regioninfo`
  - `info:server`
  - `info:serverstartcode`

  如果一个表处于切分的过程中，即region切分，还会多出两列`info:splitA`和`info:splitB`，存储值也是HRegionInfo对象，拆分结束后，删除这两列。

<font color="red">注意：</font>

在客户端对元数据进行操作的时候才会连接master，如果对数据进行读写，直接连接zookeeper读取目录/hbase/meta-region-server节点信息，会记录meta表的位置。直接读取即可，不需要访问master，这样可以减轻master的压力，相当于master专注meta表的写操作，客户端可以直接读取meta表。

在HBase的2.3版本更新了一种新模式：Master Registry。客户端可以访问master 来读取meta表信息，加大了master的压力，减轻了zookeeper的压力。

## 5.2 RegionServer架构

![image-20230626220248724](HBase2.x.assets/image-20230626220248724.png)

## 5.3 写流程

## 5.4 MemStore Flush

## 5.5 读流程

## 5.6 StoreFile Compaction

## 5.7 Region Split

# 6. HBase优化



# 7. 集成

## 7.1 HBase集成Phonix

## 7.2 HBase集成Hive



# 参考

[1] [尚硅谷HBase2.x教程（一套全面掌握hbase）](https://www.bilibili.com/video/BV1PZ4y1i7gZ/?spm_id_from=333.999.0.0&vd_source=c1dd3a0be4a3190c757c00845919b313)

