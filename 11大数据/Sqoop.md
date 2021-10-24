文档来源：[尚硅谷Sqoop教程(sqoop大数据开发标配)](https://www.bilibili.com/video/BV1jb411A7tc?spm_id_from=333.999.0.0)

# 1. Sqoop简介

[Sqoop官网](http://sqoop.apache.org/)

Sqoop是一款开源工具，主要用于在Hadoop（Hive）与传统的数据库（MySQL、PostgresSQL等）间进行数据的传递，可以将一个关系型数据库（例如：MySQL，Oracle，Postgres等）中的数据导入（`import`）到Hadoop的HDFS中，也可以将HDFS的数据导出（`export`）到关系型数据库中。

Sqoop项目开始与2009年，最早是作为Hadoop的一个第三方模块存在，后来为了让使用者能够快速部署，也为了让开发人员能够更快速地迭代开发，Sqoop独立称为一个Apache项目。

Sqoop2的最新版本是1.99.7。请注意，2与1不兼容，且特征不完整，它并不打算用于生产部署。



# 2. Sqoop原理

将导入或导出命令翻译成mapreduce程序来实现。

在翻译出的mapreduce中主要是对inputformat和outputformat进行定制。

# 3. Sqoop安装

安装Sqoop的前提是已经具备Java和Hadoop的环境。

## 3.1 下载并解压

1. 下载地址：

2. 上传安装包到虚拟机中

3. 解压sqoop安装包到指定目录，如：

   ```shel
   $ tar -zxf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /opt/module/
   ```

   

## 3.2 修改配置文件

Sqoop的配置文件与大多数大数据框架类似，在sqoop根目录下的conf目录中。

1. 重命名配置文件

   ```shell
   $ mv sqoop-env-template.sh sqoop-env.sh
   ```

   

2. 修改配置文件sqoop-env.sh

   ```shel
   export HADOOP_COMMON_HOME=/opt/module/hadoop-2.7.2
   export HADOOP_MAPRED_HOME=/opt/module/hadoop-2.7.2
   export HIVE_HOME=/opt/module/hive
   export ZOOKEEPER_HOME=/opt/module/zookeeper-3.4.10
   export ZOOCFGDIR=/opt/module/zookeeper-3.4.10
   export HBASE_HOME=/opt/module/hbase
   ```

   

## 3.3 拷贝JDBC驱动

拷贝jdbc驱动到sqoop的lib目录下，如：

```shell
$ cp mysql-connector-java-5.1.27-bin.jar /opt/module/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/lib/
```

## 3.4 验证Sqoop

我们可以通过某一个command来验证sqoop配置是否正确：

```shell
$ bin/sqoop help

```

出现一些warning警告（警告信息已省略），并伴随着帮助命令的输出：

```shell
usage: sqoop COMMAND [ARGS]

Available commands:
  codegen            Generate code to interact with database records
  create-hive-table  Import a table definition into Hive
  eval               Evaluate a SQL statement and display the results
  export             Export an HDFS directory to a database table
  help               List available commands
  import             Import a table from a database to HDFS
  import-all-tables  Import tables from a database to HDFS
  list-databases     List available databases on a server
  list-tables        List available tables in a database
  version            Display version information

See 'sqoop help COMMAND' for information on a specific command.
```

## 3.5 测试Sqoop是否能够成功连接数据库

```shell
$ bin/sqoop list-databases --connect jdbc:mysql://hadoop102:3306/ --username root --password 000000
```

出现如下输出：

```shell
information_schema
metastore
mysql
oozie
performance_schema
```



# 4. Sqoop的简单使用案例

## 4.1 导入数据

==在Sqoop中，“导入”概念指：从非大数据集群（RDBMS）向大数据集群（HDFS, HIVE, HBASE）中传输数据，即使用`import`关键字。==

### 4.1.1 RDBMS到HDFS

（1）确定M有SQL服务开启正常

（2）在MySQL中新建一张表并插入一些数据

```shell
$ mysql -uroot -p000000
mysql> create database company;
mysql> create table company.staff (id int(4) primary key not null auto_increment, name varchar(255), sex varchar(255));
mysql> insert into company.staff(name, sex) values('Thomas', 'Male');
mysql> insert into company.staff(name, sex) values('catalina', 'Female')
```



（3）导入数据

​	1）全部导入

```shell
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t"
```



​	2） 查询导入

```shell
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--query 'select name, sex from staff where id <=1 and $CONDITIONS;'
```

<span style="color:red;">提示：must contain '$CONDITIONS' in WHERE clause;</span>

<span style="color:red;">如果query后使用的是双引号，则$CONDITIONS前必须加转义符，防止shell识别为自己的变量。</span>

​	3）导入指定列

```shell
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--columns id,sex \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t"
```

<span style="color:red;">提示：columns中如果涉及到多个列，用逗号（comma）分隔，分隔时不要添加空格。</span>



​	4）使用sqoop关键字筛选查询导入数据

```shell
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--where "id=1"
```

同时指定--column和--where是可以的。

```shell
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--columns id,sex \
--where "id=1"
```

<span style="color:red;">注：不能同时指定--query和--where。</span>

### 4.1.2 导入数据到Hive

```shell
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--num-mappers 1 \
--hive-import \
--fields-terminated-by "\t" \
--hive-overwrite \
--hive-table staff_hive
```

这里其实分为两步：

第一步，将数据导入到HDFS

第二步，将HDFS中该数据，移动到hive在HDFS中的目录下，

### 4.1.3 导入数据到HBase

```shell
$ bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--columns "id,name,sex" \
--column-family "info" \
--hbase-create-table \
--hbase-row-key "id" \
--hbase-table "hbase_company" \
--num-mappers 1 \
--split-by id
```

<span style="color:red;">提示：sqoop1.4.6只支持对HBase1.0.1之前的版本的自动创建HBase表。</span>

解决方法：手动创建爱你HBase表

```shell
hbase> create 'hbase_company', 'info'
```

在HBase中scan这张表得到如下内容：

```shell
hbase> scan 'hbase_company'
```



## 4.2 导出数据

==在sqoop中，“导出”概念指：从大数据集群（HDFS，HIVE, HBASE）向非大数据集群（RDBMS）中传输数据，即使用`export`关键字。==

### 4.2.1 HIVE/HDFS 到 RDBMS

```shell
$ bin/sqoop export \
--connect jdbc:mysql://hadoop102:3306/company \
--usernae root \
--password 000000 \
--table staff \
--num-mappers 1 \
--export-dir /user/hive/warehouse/staff_hive \
--input-fields-terminated-by "\t"
```

<span style="color:red;">提示：MySQL中如果表不存在，不会自动创建。</span>

==不支持直接从HBase导出到MySQL。==

## 4.3 脚本打包

使用opt格式的文件打包sqoop命令，然后执行

（1）创建一个.opt文件

```shell
$ mkdir opt
$ touch opt/job_HDFS2RDBMS.opt
```



（2）编写sqoop脚本

```shell
$ vim opt/job_HDFS2RDBMS.opt

export
--connect
jdbc:mysql://hadoop102:3306/company
--username
root
--password
000000
--table
staff
--num-mappers
1
--export-dir
/user/hive/warehouse/staff_hive
--input-fields-terminated-by
"\t"
```

（3）执行该脚本

```shell
$ bin/sqoop --options-file opt/job_HDFS2RDBMS.opt
```



# 5. Sqoop一些常用命令及参数

## 5.1 常用命令列举

sql to hadoop

https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html

## 5.2 命令&参数详解

刚才列举了一些Sqoop的常用命令，对于不同的命令，有不同的参数，让我们来一一列举说明。

首先来我们来介绍一下公用的参数，所谓公用参数，就是大多数命令都支持的参数。

### 5.2.1 公用参数：数据库连接

| **序号** | **参数**            | **说明**               |
| -------- | ------------------- | ---------------------- |
| 1        | –connect            | 连接关系型数据库的URL  |
| 2        | –connection-manager | 指定要使用的连接管理类 |
| 3        | –driver             | Hadoop根目录           |
| 4        | –help               | 打印帮助信息           |
| 5        | –password           | 连接数据库的密码       |
| 6        | –username           | 连接数据库的用户名     |
| 7        | –verbose            | 在控制台打印出详细信息 |

### 5.2.2 公用参数：import

| **序号** | **参数**                | **说明**                                                     |
| -------- | ----------------------- | ------------------------------------------------------------ |
| 1        | –enclosed-by            | 给字段值前加上指定的字符                                     |
| 2        | –escaped-by             | 对字段中的双引号加转义符                                     |
| 3        | –fields-terminated-by   | 设定每个字段是以什么符号作为结束，默认为逗号                 |
| 4        | –lines-terminated-by    | 设定每行记录之间的分隔符，默认是\n                           |
| 5        | –mysql-delimiters       | Mysql默认的分隔符设置，字段之间以逗号分隔，行之间以\n分隔，默认转义符是\，字段值以单引号包裹。 |
| 6        | –optionally-enclosed-by | 给带有双引号或单引号的字段值前后加上指定字符。               |

### 5.2.3 公用参数：export

| **序号** | **参数**                      | **说明**                                   |
| -------- | ----------------------------- | ------------------------------------------ |
| 1        | –input-enclosed-by            | 对字段值前后加上指定字符                   |
| 2        | –input-escaped-by             | 对含有转移符的字段做转义处理               |
| 3        | –input-fields-terminated-by   | 字段之间的分隔符                           |
| 4        | –input-lines-terminated-by    | 行之间的分隔符                             |
| 5        | –input-optionally-enclosed-by | 给带有双引号或单引号的字段前后加上指定字符 |

### 5.2.4 公用参数：hive

| **序号** | **参数**                 | **说明**                                                  |
| -------- | ------------------------ | --------------------------------------------------------- |
| 1        | –hive-delims-replacement | 用自定义的字符串替换掉数据中的\r\n和\013 \010等字符       |
| 2        | –hive-drop-import-delims | 在导入数据到hive时，去掉数据中的\r\n\013\010这样的字符    |
| 3        | –map-column-hive         | 生成hive表时，可以更改生成字段的数据类型                  |
| 4        | –hive-partition-key      | 创建分区，后面直接跟分区名，分区字段的默认类型为string    |
| 5        | –hive-partition-value    | 导入数据时，指定某个分区的值                              |
| 6        | –hive-home               | hive的安装目录，可以通过该参数覆盖之前默认配置的目录      |
| 7        | –hive-import             | 将数据从关系数据库中导入到hive表中                        |
| 8        | –hive-overwrite          | 覆盖掉在hive表中已经存在的数据                            |
| 9        | –create-hive-table       | 默认是false，即，如果目标表已经存在了，那么创建任务失败。 |
| 10       | –hive-table              | 后面接要创建的hive表,默认使用MySQL的表名                  |
| 11       | –table                   | 指定关系数据库的表名                                      |



