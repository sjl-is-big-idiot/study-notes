Hudi connector 使用



## 使用范围

可以作为Source/Sink 使用.  



## DDL 定义

用作数据目的

```sql
CREATE TABLE hudi_sink
(
    uuid        VARCHAR(20) PRIMARY KEY NOT ENFORCED,
    name        VARCHAR(10),
    age         INT,
    ts          TIMESTAMP(3),
    `partition` VARCHAR(20)
) WITH (
    'connector' = 'hudi'
    , 'path' = 'hdfs://HDFS1000/data/hudi/mor'
    , 'table.type' = 'MERGE_ON_READ'  --  MERGE_ON_READ 表, 默认值为 COPY_ON_WRITE
    , 'write.tasks' = '3' -- 默认为4
    , 'compaction.tasks' = '4' -- 默认为4
    --  , 'hive_sync.enable' = 'true'  -- 默认值为false
    --  , 'hive_sync.db' = 'default'
    --  , 'hive_sync.table' = 'datagen_mor_1'
    --  , 'hive_sync.mode' = 'jdbc'
    --  , 'hive_sync.username' = ''
    --  , 'hive_sync.password' = ''
    --  , 'hive_sync.jdbc_url' = 'jdbc:hive2://172.28.1.185:7001'
    --  , 'hive_sync.metastore.uris' = 'thrift://172.28.1.185:7004'
);

```





## WITH 参数

### 通用参数

| 参数值            | 必填 | 默认值 | 描述                                                         |
| ----------------- | ---- | ------ | ------------------------------------------------------------ |
| connector         | 是   | 无     | 必须填 hudi                                                  |
| path              | 是   | 无     | 数据的存储路径.                                              |
| changelog.enabled | 否   | false  | 1. sink是否接受UPDATE_BEFORE的数据; 2. source 是否一条记录的所有变换 |



### 作为Sink的参数

| 参数值                                      | 必填 | 默认值        | 描述                                                    |
| ------------------------------------------- | ---- | ------------- | ------------------------------------------------------- |
| write.tasks                                 | 否   | 4             | 写算子的并行度                                          |
| table.type                                  | 否   | COPY_ON_WRITE | hudi 表类型. 可选值为 COPY_ON_WRITE 或者 MERGE_ON_READ  |
| hoodie.datasource.write.recordkey.field     | 否   | uuid          | 记录的key字段                                           |
| write.precombine.field                      | 否   | ts            | 相同key记录时候, 用于比较的字段.                        |
| hoodie.datasource.write.partitionpath.field | 否   | 无            |                                                         |
|                                             |      |               |                                                         |
| compaction.tasks                            | 否   | 4             | 合并任务的并行度                                        |
| compaction.async.enabled                    | 否   | true          |                                                         |
| compaction.trigger.strategy                 | 否   | num_commits   | num_commits / time_elapsed / num_and_time / num_or_time |
|                                             |      |               |                                                         |
| hive_sync.enable                            | 否   | false         |                                                         |



### 作为Source的参数

| 参数值                         | 必填 | 默认值   | 描述                                                         |
| ------------------------------ | ---- | -------- | ------------------------------------------------------------ |
| read.tasks                     | 否   | 4        | 读算子的并行度                                               |
| hoodie.datasource.query.type   | 否   | snapshot | 可选值 snapshot / read_optimized / incremental               |
| read.streaming.enabled         | 否   | false    |                                                              |
| read.streaming.check-interval  | 否   | 60       | 单位秒.  streaming read的检查时间间隔                        |
| read.streaming.skip_compaction | 否   | false    |                                                              |
| read.start-commit              | 否   | 无       | 格式为 yyyyMMddHHmmss; 可设置为 earliest, 表示从最早的commit开始消费 |
| read.end-commit                | 否   | 无       | streaming read,无需设置该值                                  |



### 更详细的参数配置

可以参见 [Flink Options](https://hudi.apache.org/docs/0.10.1/configurations/#FLINK_SQL)



## COS 配置

无需做额外配置.

path 填写为对应的 cosn 路径即可.



## HDFS 配置

### 获取 HDFS 链接配置jar

Flink SQL 任务写 Hudi , 使用 HDFS 存储时需要使用包含  HDFS 配置信息的 jar 包来连接到 HDFS 集群。具体获取连接配置 jar 及其使用的步骤如下：

1. ssh 登录到对应 HDFS 集群节点。

2. 获取 hdfs-site.xml，EMR 集群中的配置文件在如下位置。 							 						

   ```
   /usr/local/service/hadoop/etc/hadoop/hdfs-site.xml
   ```

3. 对获取到的配置文件 打 jar 包。

   ```bash
   jar -cvf hdfs-xxx.jar hdfs-site.xml
   ```


4. 校验 jar 的结构（可以通过 vi 命令查看 ），jar 里面包含如下信息，请确保文件不缺失且结构正确。

   ```
   vi hdfs-xxx.jar
   ```

   ​								 							 							 						

   ```bash
   META-INF/
   META-INF/MANIFEST.MF
   hdfs-site.xml
   ```

### 在任务中使用配置 jar

引用程序包中选择连接配置 jar 包（该 jar 包为在上述中得到的 hdfs-xxx.jar，必须在依赖管理上传后才使用）。

>  Flink 作业默认以 flink 用户操作 HDFS，若没有 HDFS 路径的写入权限，可通过作业 [高级参数](https://cloud.tencent.com/document/product/849/53391) 设置为有权限的用户，或者设置为超级用户 hadoop。
>
> ​								 							 							 						
>
> ```
> containerized.taskmanager.env.HADOOP_USER_NAME: hadoop
> containerized.master.env.HADOOP_USER_NAME: hadoop
> ```



## Kerberos 认证授权

1. 登录集群 Master 节点，获取 krb5.conf、emr.keytab、core-site.xml、hdfs-site.xml 文件，路径如下。

   ​									 							 						

   ```
   /etc/krb5.conf
   /var/krb5kdc/emr.keytab
   /usr/local/service/hadoop/etc/hadoop/core-site.xml
   /usr/local/service/hadoop/etc/hadoop/hdfs-site.xml
   ```

2. 对获取的配置文件打 jar 包。

   ​								 							 							 						

   ```
   jar cvf hdfs-xxx.jar krb5.conf emr.keytab core-site.xml hdfs-site.xml 
   ```

2. 校验 jar 的结构（可以通过 vim 命令查看 vim hdfs-xxx.jar），jar 里面包含如下信息，请确保文件不缺失且结构正确。

   ​								 							 							 						

   ```
   META-INF/
   META-INF/MANIFEST.MF
   emr.keytab
   krb5.conf
   hdfs-site.xml
   core-site.xml
   ```
   
3. 在 [程序包管理](https://console.cloud.tencent.com/oceanus/resource) 页面上传 jar 包，并在作业参数配置里引用该程序包。

4. 获取 kerberos principal，用于作业 [高级参数](https://cloud.tencent.com/document/product/849/53391) 配置。

   ​								 							 							 						

   ```
   klist -kt /var/krb5kdc/emr.keytab
   
   # 输出如下所示，选取第一个即可：hadoop/172.28.28.51@EMR-OQPO48B9
   KVNO Timestamp     Principal
   ---- ------------------- ------------------------------------------------------
     2 08/09/2021 15:34:40 hadoop/172.28.28.51@EMR-OQPO48B9 
     2 08/09/2021 15:34:40 HTTP/172.28.28.51@EMR-OQPO48B9 
     2 08/09/2021 15:34:40 hadoop/VM-28-51-centos@EMR-OQPO48B9 
     2 08/09/2021 15:34:40 HTTP/VM-28-51-centos@EMR-OQPO48B9 
   ```
   
5. 作业 [高级参数](https://cloud.tencent.com/document/product/849/53391) 配置。

   ​								 							 							 						

   ```
   containerized.taskmanager.env.HADOOP_USER_NAME: hadoop
   containerized.master.env.HADOOP_USER_NAME: hadoop
   security.kerberos.login.principal: hadoop/172.28.28.51@EMR-OQPO48B9
   security.kerberos.login.keytab: emr.keytab
   security.kerberos.login.conf: krb5.conf.path
   ```



