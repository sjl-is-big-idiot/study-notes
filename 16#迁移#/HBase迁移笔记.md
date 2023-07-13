

![img](HBase迁移笔记.assets/8bd3437f00d8ee078449d393e167c437.png)



# 1. snapshot+replication方式

## 1.1 snapshot

HBase snapshot只需消耗很少的性能就能对table做内容和元数据的copy。snapshot是不可改变的，它包含**table的元数据信息**、做快照时刻此表的**HFile列表**。无论表时enable还是disable状态都可以做快照，**快照操作不涉及底层数据的拷贝**。

- clone快照是根据快照创建了一个新表。
- restore快照是将表的内容恢复到创建此快照的时刻的内容。

clone和restore操作不需要进行任何数据拷贝，因为table底层的HFile文件不会因为做了这两个操作而发生改变。

**0.94.6之前要对HBase的表做backup或clone**，只能使用：

- **CopyTble**/**ExportTable**

  **缺点**是会严重影响RegionServer的性能。

- **disable表之后，拷贝该表的HFile文件**

  缺点是因为disable了表，导致期间无法读写此表。

要对table做snapshot如需要保证`hbase-site.xml`中参数`hbase.snapshot.enabled`为`true`。0.94.5以后默认为true。

```xml
  <property>
    <name>hbase.snapshot.enabled</name>
    <value>true</value>
  </property>
```

### 1.1.1 **做快照**

```bash
$ ./bin/hbase shell
# 1、默认的snapshot，会将当前memstore中的写数据flush（刷盘）之后再做快照，目的保证做出的快照中也包含此时刻内存中的数据。
hbase> snapshot 'myTable', 'myTableSnapshot-122112'

# 2、在snapshot时不将当前memstore中的数据flush（刷盘）
hbase> snapshot 'mytable', 'snapshot123', {SKIP_FLUSH => true}

# 3、打一个带TTL的快照，快照的生命周期与表的生命周期相互独立，TTL单位秒，TTL<-1也表示永久保存此snapshot
hbase> snapshot 'mytable', 'snapshot1234', {TTL => 86400}

# 4、设置快照的MAX_FILESIZE，当根据此快照clone出表之后，会在表的元数据中添加MAX_FILESIZE=此值，对应的配置
#    项为hbase.hregion.max.filesize，代表HFile的最大大小，超过则会切分。
#		当在hbase快照迁移中，如果源hbase集群的hbase.hregion.max.filesize源大于目的，那么会导致打的快照对应的HFile会很大
#   结果是在目标端需要对这些HFile做切分，可能会出现region切分的风暴
snapshot 'table01', 'snap01', {MAX_FILESIZE => 21474836480}

# 5、启用/禁用快照自动清理，默认是启用的。
	# 禁用
	hbase> snapshot_cleanup_switch false
	# 启用
	hbase> snapshot_cleanup_switch true
	# 查看是否启用了快照自动清理
	snapshot_cleanup_enabled
```

目前无法确定打的快照中是否包含memstore中的数据。快照TTL相关参数`hbase.master.snapshot.ttl`，若配置了，则作为快照的默认TTL；若未配置则默认snapshot永不过期。

**快照的生命周期**

- 创建快照时，显式指定TLL，若TTL<-1,永久保存此snapshot

- 创建快照时，显式指定TLL，若TTL > 0，则指定秒数之后，自动删除此snapshot（前提是启用了快照自动清理）

- 创建快照时，没有显式指定TTL的使用`hbase.master.snapshot.ttl（默认0，永不过期）`作为快照的TTL。

  

Since Replication works at log level and snapshots at file-system level, after a restore, the replicas will be in a different state from the master. If you want to use restore, you need to stop replication and redo the bootstrap.

### 1.2 拷贝快照到新集群

**ExportSnapshot 工具**可以将snapshot相关的数据（HFiles、logs、snapshot元数据）拷贝到其他集群。ExportSnapshot工具执行MapReduce job来将文件从原集群拷贝到新集群，属于**HDFS文件级别的操作**。

```bash
# 
$ bin/hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot MySnapshot -copy-to hdfs://srv2:8082/hbase -mappers 16 -bandwidth 200
```

- `-snapshot` 指定要复制的快照名
- `-copy-to` 指定快照文件复制到哪个HDFS路径
- `-mappers`指定使用多少个线程来进行文件的拷贝
- `-bandwidth` 指定每个mapper在文件复制所用的带宽，如200MB/s

## 2. replication

HBase提供了replication的方式，可以在将A集群的状态同步至B集群。原理是使用源端集群的WAL来传递状态的变化给其他集群。

replication是列族级别的复制

前提：

- 在启用replication之前，需要在目标HBase集群创建replication的表和replication的列族。
- 源目HBase集群的机器网络互通
- 如果源目HBase集群使用同一个zk集群，则`zookeeper.znode.parent`应为不同的值。
- 在源集群使用`add_peer`添加目标集群为peer。
- 在源集群使用`enable_table_replication`启用HBase表的replication。



查看replication状态

```bash
status 'replication'
```





