海豚官网：https://dolphinscheduler.apache.org/zh-cn

# 1. 概述

## 1.1 DolphinScheduler是什么？

Apache DolphinScheduler是一个分布式和可扩展的开源工作流协调平台，具有强大的DAG可视化界面。

## 1.2 DolphinScheduler能干什么？

Apache DolphinScheduler 旨在**解决复杂的大数据任务依赖关系，并为应用程序提供数据和各种 OPS 编排中的关系。 解决数据研发ETL依赖错综复杂，无法监控任务健康状态的问题。** DolphinScheduler 以 DAG（Directed Acyclic Graph，DAG）流式方式组装任务，可以及时监控任务的执行状态，支持重试、指定节点恢复失败、暂停、恢复、终止任务等操作。

## 1.3 特性

- **简单易用**
  - **可视化 DAG**: 用户友好的，通过拖拽定义工作流的，运行时控制工具
  - **模块化操作**: 模块化有助于轻松定制和维护。
- **丰富的使用场景**
  - **支持多种任务类型**: 支持Shell、MR、Spark、SQL等10余种任务类型，支持跨语言，易于扩展
  - **丰富的工作流操作**: 工作流程可以定时、暂停、恢复和停止，便于维护和控制全局和本地参数。

- **High Reliability**
  - **高可靠性**: 去中心化设计，确保稳定性。 原生 HA 任务队列支持，提供过载容错能力。 DolphinScheduler 能提供高度稳健的环境。

- **High Scalability**
  - **高扩展性**: 支持多租户和在线资源管理。支持每天10万个数据任务的稳定运行。

**DolphinScheduler默认的网络端口配置**

| 组件                 | 默认端口 | 说明                               |
| -------------------- | -------- | ---------------------------------- |
| MasterServer         | 5678     | 非通信端口，只需本机端口不冲突即可 |
| WorkerServer         | 1234     | 非通信端口，只需本机端口不冲突即可 |
| ApiApplicationServer | 12345    | 提供后端通信端口                   |

# 2. 安装

支持4种方式安装：

- **单机部署（Standalone）**

  仅用于学习和体验。

  ***注意:\*** Standalone仅建议20个以下工作流使用，因为其采用内存式的H2 Database, Zookeeper Testing Server，任务过多可能导致不稳定，并且**如果重启或者停止standalone-server会导致内存中数据库里的数据清空**。 如果您要连接外部数据库，比如mysql或者postgresql，请看[配置数据库](https://dolphinscheduler.apache.org/zh-cn/docs/3.1.4/guide/installation/standalone#配置数据库)

- **伪集群部署（Pseudo-Cluster）**

  伪集群部署目的是在单台机器部署 DolphinScheduler 服务，**该模式下master、worker、api server 都在同一台机器上**

- **集群部署（Cluster）**

- **kubernetes部署**

## 2.1 下载

官方下载地址：https://dolphinscheduler.apache.org/zh-cn/download/3.1.4

华为云镜像地址：https://repo.huaweicloud.com/apache/dolphinscheduler/

## 2.2 单机部署

TODO

## 2.3 伪集群部署

TODO

## 2.4 集群部署

### 2.4.1 前置准备工作

伪分布式部署 DolphinScheduler 需要有外部软件的支持

- JDK：下载[JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html) (1.8+)，安装并配置 `JAVA_HOME` 环境变量，并将其下的 `bin` 目录追加到 `PATH` 环境变量中。如果你的环境中已存在，可以跳过这步。
- 二进制包：在[下载页面](https://dolphinscheduler.apache.org/zh-cn/download)下载 DolphinScheduler 二进制包
- 数据库：[PostgreSQL](https://www.postgresql.org/download/) (8.2.15+) 或者 [MySQL](https://dev.mysql.com/downloads/mysql/) (5.7+)，两者任选其一即可，如 MySQL 则需要 JDBC Driver 8.0.16
- 注册中心：[ZooKeeper](https://zookeeper.apache.org/releases.html) (3.4.6+)，[下载地址](https://zookeeper.apache.org/releases.html)
- 进程树分析
  - macOS安装`pstree`
  - Fedora/Red/Hat/CentOS/Ubuntu/Debian安装`psmisc`

> ***注意:\*** DolphinScheduler 本身不依赖 Hadoop、Hive、Spark，但如果你运行的任务需要依赖他们，就需要有对应的环境支持

### 2.4.2 环境准备

#### 2.4.2.1 配置用户sudo免密及权限

创建部署用户，并且一定要配置 `sudo` 免密。以创建 dolphinscheduler 用户为例

```shell
# 创建用户需使用 root 登录
useradd dolphinscheduler

# 添加密码
echo "dolphinscheduler" | passwd --stdin dolphinscheduler

# 配置 sudo 免密
sed -i '$adolphinscheduler  ALL=(ALL)  NOPASSWD: NOPASSWD: ALL' /etc/sudoers
sed -i 's/Defaults    requirett/#Defaults    requirett/g' /etc/sudoers

# 修改目录权限，使得部署用户对二进制包解压后的 apache-dolphinscheduler-*-bin 目录有操作权限
chown -R dolphinscheduler:dolphinscheduler apache-dolphinscheduler-*-bin
```

> ***注意:\***
>
> - 因为任务执行服务是以 `sudo -u {linux-user}` 切换不同 linux 用户的方式来实现多租户运行作业，所以部署用户需要有 sudo 权限，而且是免密的。初学习者不理解的话，完全可以暂时忽略这一点
> - 如果发现 `/etc/sudoers` 文件中有 "Defaults requirett" 这行，也请注释掉

#### 2.4.2.2 配置机器SSH免密登陆

由于安装的时候需要向不同机器发送资源，所以要求各台机器间能实现SSH免密登陆。配置免密登陆的步骤如下

```shell
su dolphinscheduler

ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

> ***注意:\*** 配置完成后，可以通过运行命令 `ssh localhost` 判断是否成功，如果不需要输入密码就能ssh登陆则证明成功

### 2.4.2.3 启动zookeeper

DolphinScheduler的部署依赖于ZooKeeper，ZooKeeper的安装见对应的文档。

在每个ZooKeeper机器上启动ZooKeeper进程，进入ZooKeeper的安装目录

```bash
./bin/zkServer.sh start
```

### 2.4.5 修改DolphinScheduler配置

- install_env.sh

  描述了哪些机器将被安装DolphinScheduler，每台机器对应的安装哪些服务。文件路径为`<DolphinScheduler安装目录>/bin/env/install_env.sh`

- dolphinscheduler_env.sh

  文件路径为`<DolphinScheduler安装目录>/bin/env/dolphinscheduler_env.sh`

#### 2.4.5.1 修改install_env.sh文件

#### 2.4.5.2 修改dolphinscheduler_env.sh文件





## 2.5 kubernetes部署

TODO





# 3. 常用命令

启停命令

```bash
# 一键停止集群所有服务
bash ./bin/stop-all.sh

# 一键开启集群所有服务
bash ./bin/start-all.sh

# 启停 Master
bash ./bin/dolphinscheduler-daemon.sh stop master-server
bash ./bin/dolphinscheduler-daemon.sh start master-server

# 启停 Worker
bash ./bin/dolphinscheduler-daemon.sh start worker-server
bash ./bin/dolphinscheduler-daemon.sh stop worker-server

# 启停 Api
bash ./bin/dolphinscheduler-daemon.sh start api-server
bash ./bin/dolphinscheduler-daemon.sh stop api-server

# 启停 Alert
bash ./bin/dolphinscheduler-daemon.sh start alert-server
bash ./bin/dolphinscheduler-daemon.sh stop alert-server
```



# 4. 使用

