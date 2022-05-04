# 前言

YARN的发展阶段

#### 1.0 阶段-1：Ad Hoc集群出现前

> 在少量几个节点上手工建立一个集群，将数据载入HDFS中，通过运行MapReduce任务来计算并获的感兴趣的结果，然后拆掉集群。这一时期，并没有在HDFS中持久化存储数据，因为当时还没有这种迫切需求。

#### 1.1 阶段0：Ad Hoc集群时期

> 在上一个阶段的基础上，实现了持久化的HDFS集群。Yahoo安装并运营了一个共享的HDFS实例，并将一些数据集载入其中，供他人使用。这一时期，支持多用户环境，文件和命名空间配额，以及其他改进多租户操作的特性。

#### 1.2 阶段1：Hadoop on Demand(HOD)

> 在商用硬件的共享集群上提供和管理MapReduce和HDFS实例的系统。HOD使用传统的资源管理器（Torque）和集群调度器（Maui）一起来分配共享节点池上的Hadoop集群。

![在这里插入图片描述](YARN.assets/20210424182428177.png)

一个典型的HOD用户会话过程如下：

1. 用户调用HOD Shell，向Torque提交一个适当规模的计算集群描述请求。

2. Torque会让请求排队等待，直到有节点可用。一旦节点可用，Torque在其中一个计算几点上启动名为RingMaster的首进程。

3. RingMaster是一个HOD组件，它通过ResourceManager接口来运行HODRing组件，每个分配的计算节点中都包含HODRing。

4. HODRing初始化之后，会与RingMaster通信以获取Hadoop命令，并遵照执行。一旦Hadoop的守护进程开始启动，HODRing会向RingMaster注册，提供关于守护进程的信息。

5. HOD客户端保持与RingMaster的通信，找出JobTracker和HDFS守护进程所在的位置。

6. 一旦一切就绪，用户知道了JobTracker和HDFS的位置，HOD会退出并允许用户在相应的集群上执行他的数据运算。

7. 当运行完数据运算作用后，用户即可释放集群。

   

优点是：

1. 可维持性好；

2. 日志管理功能；

3. 多用户和单用户多集群功能；

4. 配置机制方便；

5. 空闲集群的自动释放。

   

缺点是：

1. 没有考虑到数据本地化，即不支持位置感知（将计算移动到数据所在的位置）；
2. 集群使用率不高。

#### 1.3 阶段2：共享计算集群的黎明

属于Apache Hadoop 1.x。此阶段为共享MapReduce集群阶段。

> 由于HOD架构掌握的信息太少，无法对分配作出智能决策。它的资源粒度太粗糙等缺点。因此发展为包含共享HDFS实例的MapReduce集群模型。
>
> JobTraker：一个中央守护进程，负责运行集群上的所有作业。
>
> TaskTracker：系统里的从进程，根据JobTracker的指令来执行任务。

优点是：

1. 中央JobTracker守护进程
2. JobTracker内存管理
3. 已完成作业的管理
4. 中心调度器
5. 恢复和升级
6. 单个节点上的隔离
7. 安全性
8. 身份认证和访问控制
9. 各种集群管理功能

缺点是：

1. 扩展性瓶颈
2. 可靠性和可用性不足
3. 对编程模型多样性的支持，仅支持MapReduce，难以扩展到其他编程模型。
4. 用户日志管理
5. 敏捷性不够好

#### 1.4 阶段3：YARN的出现

> 前一个阶段的JobTracker扩展性不好，需要重写，以及其他不足之处，促使YARN的诞生。
>
> YARN的全称是Yet Another Resource Negotiator
>
> 优点是：
>
> 1. 可扩展性
>
> 2. 可维持性
>
> 3. 多租户
>
> 4. 位置感知
>
> 5. 集群使用率高
>
> 6. 安全和可审计的操作
>
> 7. 可靠性和可用性
>
> 8. 对编程模型多样性的支持
>
> 9. 灵活的资源模型
>
> 10. 向后兼容
>
>     

YARN的工作流程大约如下：

![在这里插入图片描述](YARN.assets/20210424182541223.png)

![在这里插入图片描述](YARN.assets/20210424182541192.png)

可以认为是，将调度和资源管理从第1版中的MapReduce中分离出来了，成为了YARN。

# 安装

分为单机部署、伪分布式部署、和分布式部署。

# 核心组件和概念

YARN的工作流程图如下：



## ResourceManager

ResourceManager，主要职责是调度，即它在竞争的应用程序之间分配系统中的可用资源，但不关注每个应用程序的状态管理（有AppolicationMaster负责）。为了适应不同的策略，RM有一个可插拔的调度器来应用不同的算法，Hadoop 2.x中支持3中调度器：`FIFO`、`Capacity`、`Fair`。

## NodeManager

管理Hadoop集群中独立的计算节点，负责管理和监控节点和Container。职责包括：

1. 与ResourceManager保持通信
2. 管理Container的生命周期，监控每个Container的资源使用（内存，CPU）情况
3. 跟踪节点健康状况等。
4. 负责资源本地化，即负责安全地下载和组织Contianer所需的各种文件资源，它会尽可能地将文件分散到各个可用磁盘上。有3种类型的本地化：PUBLIC资源的本地化、PRIVATE/APPLICATION资源的本地化

本地化的过程如下图所示：

![在这里插入图片描述](YARN.assets/20210424182541251.png)

## ApplicationMaster

ApplicationMaster，一个应用会启动一个ApplicationMaster（通常称之为Container0），它的职责是与ResourceManager协商资源（Container），并好NodeManager协同工作来执行和监控任务。

## Contianer

Contianer，是对资源的抽象，是单个节点上如RAM、CPU核和磁盘等物理资源的集合。单个节点上可以有多个Container。可以修改Container的内存、CPU大小。每个节点可以看做由多个Container构成。

Container代表了集群中单个节点上的一组资源（内存，CPU），由NodeManager监控，由ResourceManager调度。在运行过程中可以动态的请求或释放Container。

资源模型，一个应用可以通过ApplicationMaster请求非常具体的资源，如：

```shell
1. 资源名称
2. 内存量
3. CPU（核数/类型）
4. 其他资源，如`disk/network I/O`、GPU等
```

## Scheduler

Scheduler，可插拔的调度器组件，支持FIFO（先入先出），Capacity和Fair。

1. FIFO调度器，即”先来先服务“，不考虑作业的优先级和范围。适合低负载集群，不适合大型的共享集群。

2. Capacity调度器，容量调度，配置一个或多个队列，保证每个队列的最小资源使用值。通过ACL（访问控制列表）用来控制哪些用户可以向各个队列提交作业。多余的容量会优先分配给那些最饥饿的（最满的）队列。在每个队列内部使用层次化的FIFO来调度多个应用程序。

3. Fair调度器，公平调度，将资源公平的分配给应用，使得所有应用在平均情况下随着时间得到相等份额的资源。[什么是公平调度器（Fair Scheduler）？](https://blog.csdn.net/miachen520/article/details/116559553)

   ![img](YARN.assets/20210509085755197.png)

   ![img](YARN.assets/20210509085826980.png)

**FIFO、Capacity、Fair调度器对比**

[Hadoop的三种调度器](https://www.cnblogs.com/zhipeng-wang/p/14480138.html)





- FIFO：在FIFO 调度器中，只根据应用入队的先后顺序来分配资源。就像上厕所排队一样。

  ![img](YARN.assets/1651153-20200331141247941-750726635.png)

  ​																			图片来自互联网

  - 优点：简单
  - 缺点：大的应用可能会占用所有集群资源，小应用就会被大应用阻塞。

- Capacity：各自使用自己的队列，队列有资源配置限制。就像男女厕所排队一样，男厕位置很空，女厕爆满，但是无法利用男厕的资源。

  ![img](YARN.assets/1651153-20200331141910222-1137004991.png)

  ​																			图片来自互联网

  - 优点：实现了资源的隔离，避免不同队列中的应用相互影响。
  - 缺点：queue1没有应用，queue2应用超多资源已经跑满了，但是queue2中排队的应用是不能够使用queue1中的空闲资源的。资源存在浪费的情况。

- Fair：弹性队列，可占用一部分其他队列的资源。

  ![img](YARN.assets/1651153-20200331141347663-1699272976.png)

  ​															图片来自互联网

  - 优点：
    - Fair Scheduler队列内部支持多种调度策略，包括FIFO、Fair（队列中的N个作业，每个获得该队列1 / N的资源）、DRF（Dominant Resource Fairness）（多种资源类型e.g. CPU，内存 的公平资源分配策略）
    - 支持资源抢占，可以有效地利用集群中的资源，
    - air Scheduler支持资源抢占。当队列中有新的应用提交时，系统调度器理应为它回收资源，但是考虑到共享的资源正在进行计算，所以调度器采用先等待再强制回收的策略，即等待一段时间后如果仍没有获得资源，那么从使用共享资源的队列中杀死一部分任务，通过yarn.scheduler.fair.preemption设置为true，开启抢占功能。
    - Fair Scheduler可以为每个队列单独设置调度策略（FIFO Fair DRF）
  - 缺点：相对来说比capacity复杂





## YARN架构

Hadoop 1和Hadoop2架构对比

![在这里插入图片描述](YARN.assets/20210424182541239.png)

YARN的架构图如下：

![在这里插入图片描述](YARN.assets/20210424182541262.png)

## 工作流程

工作流程在上面也提到过多次，大致如下图所示：

![在这里插入图片描述](YARN.assets/20210424182541278.png)

![在这里插入图片描述](YARN.assets/20210424182541223.png)

![在这里插入图片描述](YARN.assets/20210424182541238.png)

1. 客户端提交application请求
2. 应答带回ApplicationID
3. Application Submission Context包含ApplicationID，用户名，队列名等，及Container Launched Context中的资源请求，作业文件，安全令牌等。
4. 启动应用对应的ApplicationMaster，并向ResourceManager发送注册请求
5. 资源容量请求
6. ApplicationMaster向ResourceManager请求Container
7. ResourceManager分配Container给ApplicationMaster
8. ApplicationMaster传递Container给NM来启动Container
9. ApplicationMaster请求Container的状态，Container响应状态信息给ApplicationMaster

YARN作业的运行图，如下所示：

![img](YARN.assets/1691221-20200215101916053-359730273.png)



[Hadoop yarn工作流程详解](https://www.cnblogs.com/Transkai/p/10549923.html)

![img](YARN.assets/1598493-20190318004612164-202277806.png)

# 参考资料

[1] 《Hadoop-YARN权威指南》

