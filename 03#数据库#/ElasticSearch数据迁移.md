# 1. 目标

将源端es中的所有数据都迁移到目标端。

源端es数据量约2G左右。

# 2. 方案

- snapshot方式
- logstash方式
- elasticdump方式

# 3. 测试

## 3.3 elasticdump方式

### 3.3.1 安装elasticdump

elasticdump依赖于nodejs，因此我们要先安装nodejs

安装nodejs

```shell

```

安装elasticdump

```shell

```

### 3.3.2 使用elaticdump进行数据迁移

安装elasticdump的机器需要同时与源端es、目标端es网络打通。

```shell

```

### 3.3.3 elasticdump数据迁移效率

数据量：

迁移时间：

带宽：

### 3.3.4 源端es和目标端es数据一致性校验

TODO



# 参考文献