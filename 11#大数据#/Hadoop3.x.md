Hadoop3.x和Hadoop2.x的区别

# 1. Hadoop概念

TODO

# 2. Hadoop安装和配置

## 2.1 单节点

TODO

## 2.2 伪分布式

TODO





## 2.3 非HA 完全分布式

步骤：

### 2.3.1 准备3台服务器

这里是用的VM Ware克隆了3台Centos7.X的VM。

1. 克隆虚拟机

   右键>管理>克隆


   `/etc/udev/rules.d/70-persistent-ipoib.rules`这个文件是关于网卡的。当网卡变动时，这个文件都会发生变化。

2. 修改克隆虚拟机的静态IP

   ```shell
   vim /etc/sysconfig/network-scripts/ifcfg-eth0
   ```

3. 修改主机名

   ```shell
   vim /etc/sysconfig/network
   vim /etc/hostname
   reboot
   ```

4. 关闭防火墙

   ```shell
   systemctl status firewalld
   systemctl stop firewalld
   systemctl status iptables
   systemctl stop iptables
   # 关闭开机自启
   systemctl disable firewalld
   systemctl disable iptables
   ```

5. 创建`atguigu`用户

   ```shell
   useradd atguigu -p sjl1991
   ```

6. 配置`atguigu`用户具有`root`权限

   ```shell
   vim /etc/sudoers
   ```

7. 在`/opt`目录下创建文件夹

   ```shell
   mkdir /opt/module
   mkdir /opt/software
   ```

8. 修改`/etc/host`

   增加如下几行。

   ```bash
   hadoop322-node01  192.168.61.129
   hadoop322-node02  192.168.61.135
   hadoop322-node03  192.168.61.136
   192.168.61.129 hadoop322-node01
   192.168.61.135 hadoop322-node02
   192.168.61.136 hadoop322-node03
   ```

   

### 2.3.2 安装和配置NTP服务

时间同步的方式：找一个机器作为时间服务器，所有的机器都与这台机器进行时间同步，比如：每隔10min同步一次（通过crontab来实现）。

![image-20210615180835140](Hadoop3.x.assets/image-20210615180835140.png)

##### 2.3.2.1 时间服务器配置（**必须root用户**）f

1. 检查ntp是否安装

   ```shell
   [atguigu@hadoop322-node01 hadoop-3.2.2]$ crontab -l
   no crontab for atguigu
   [atguigu@hadoop322-node01 hadoop-3.2.2]$ su root
   Password: 
   [root@hadoop322-node01 hadoop-3.2.2]# rpm -qa |grep ntp
   fontpackages-filesystem-1.44-8.el7.noarch
   python-ntplib-0.3.2-1.el7.noarch
   ntp-4.2.6p5-29.el7.centos.x86_64
   ntpdate-4.2.6p5-29.el7.centos.x86_64
   ```

   如果未安装ntp，使用如下命令进行安装

   ```bash
   yum install ntp -y
   ```

   

2. 修改ntp配置文件

   ```shell
   [root@hadoop322-node02 hadoop-3.2.2]# vim /etc/ntp.conf
   # 修改内容如下：
   
   # 修改1. 授权192.168.10-192.168.1.255 网段上的所有机器可以从这台机器上查询和同步时间，取消注释
   restrict 192.168.61.0 mask 255.255.255.0 nomodify notrap
   # 修改2. 集群在局域网中，不适用其他互联网上的时间，所以都注释掉
   #server 0.centos.pool.ntp.org iburst
   #server 1.centos.pool.ntp.org iburst
   #server 2.centos.pool.ntp.org iburst
   #server 3.centos.pool.ntp.org iburst
   
   # 添加3.当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步
   server 127.127.1.0						#如果国际通用时间服务器不能同步，则自动会按照本级时间进行同步
   fudge 127.127.1.0 stratum 10	#指定127.127.1.0 为第10层。ntp和127.127.1.0同步完后，就变成了11
   logfile /var/log/ntp.log　　　 #配置日志目录
   ```

3. 修改`/etc/sysconfig/ntpd`文件

   ```shell
   [root@hadoop322-node01 hadoop-3.2.2]# vim /etc/sysconfig/ntpd
   # 添加如下内容，让硬件时间与系统时间一起同步
   SYNC_HWCLOCK=yes
   ```

4. 重新启动`ntpd`服务

   ```shell
   service ntpd status
   service ntpd start
   service ntpd stop
   service ntpd restart
   
   [root@hadoop322-node01 subdir0]# service ntpd status
   Redirecting to /bin/systemctl status ntpd.service
   ● ntpd.service - Network Time Service
      Loaded: loaded (/usr/lib/systemd/system/ntpd.service; disabled; vendor preset: disabled)
      Active: inactive (dead)
   [root@hadoop322-node01 subdir0]# service ntpd start
   Redirecting to /bin/systemctl start ntpd.service
   [root@hadoop322-node01 subdir0]# service ntpd status
   Redirecting to /bin/systemctl status ntpd.service
   ● ntpd.service - Network Time Service
      Loaded: loaded (/usr/lib/systemd/system/ntpd.service; disabled; vendor preset: disabled)
      Active: active (running) since Tue 2021-06-15 18:31:45 CST; 4s ago
     Process: 10615 ExecStart=/usr/sbin/ntpd -u ntp:ntp $OPTIONS (code=exited, status=0/SUCCESS)
    Main PID: 10618 (ntpd)
       Tasks: 1
      Memory: 1.4M
      CGroup: /system.slice/ntpd.service
              └─10618 /usr/sbin/ntpd -u ntp:ntp -g
   
   Jun 15 18:31:45 hadoop322-node01 ntpd[10618]: Listen normally on 5 docker_gwbridge 172.19.0.1 ...123
   Jun 15 18:31:45 hadoop322-node01 ntpd[10618]: Listen normally on 6 br-9f19cd8b27c1 172.18.0.1 ...123
   Jun 15 18:31:45 hadoop322-node01 ntpd[10618]: Listen normally on 7 docker0 172.17.0.1 UDP 123
   Jun 15 18:31:45 hadoop322-node01 ntpd[10618]: Listen normally on 8 lo ::1 UDP 123
   Jun 15 18:31:45 hadoop322-node01 ntpd[10618]: Listen normally on 9 ens33 fe80::20c:29ff:fed3:2...123
   Jun 15 18:31:45 hadoop322-node01 ntpd[10618]: Listening on routing socket on fd #26 for interf...tes
   Jun 15 18:31:45 hadoop322-node01 ntpd[10618]: 0.0.0.0 c016 06 restart
   Jun 15 18:31:45 hadoop322-node01 ntpd[10618]: 0.0.0.0 c012 02 freq_set kernel 0.000 PPM
   Jun 15 18:31:45 hadoop322-node01 ntpd[10618]: 0.0.0.0 c011 01 freq_not_set
   Jun 15 18:31:46 hadoop322-node01 ntpd[10618]: 0.0.0.0 c514 04 freq_mode
   Hint: Some lines were ellipsized, use -l to show in full.
   ```

5. 设置`ntpd`服务开机启动

   ```shell
   chkconfig ntpd on
   ```

##### 2.3.2.2 其他机器配置（必须root用户）

1. 在其它机器配置10min与时间服务器同步一次

   ```shell
   crontab -e
   # 添加
   */10 * * * * /usr/sbin/ntpdate hadoop322-node01
   ```

2. 修改任意机器时间

   ```shell
   date -s "2020 20:20:20"
   ```

3. 十分钟后，查看机器是否与时间服务器同步

   ```shell
   date
   ```


### 2.3.3 配置集群节点免密SSH

配置SSH

```shell
[atguigu@hadoop322-node01 hadoop-3.2.2]$ ll ~/.ssh
total 4
-rw-r--r--. 1 atguigu atguigu 370 Jun 15 12:58 known_hosts
# ~/.ssh/known_hosts 记录了所有ssh访问过的主机


# 生成SSH公钥和私钥
[atguigu@hadoop322-node01 hadoop-3.2.2]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/atguigu/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/atguigu/.ssh/id_rsa.
Your public key has been saved in /home/atguigu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:pwDfVu6zXlyVg1L/Wk/Z8aj9rJL8YISOhJIWSvQ9qtM atguigu@hadoop102
The key's randomart image is:
+---[RSA 2048]----+
|  .          .   |
| . . .      . o .|
|  . + o   .. . =.|
| . . * + o ..  oB|
|  . = + S + . o.*|
|   + . + * o + +.|
|  o E   o +.*.o .|
|   .       =+. o |
|         .o  oo.o|
+----[SHA256]-----+
[atguigu@hadoop322-node01 hadoop-3.2.2]$ ll ~/.ssh
total 12
-rw-------. 1 atguigu atguigu 1675 Jun 15 14:53 id_rsa
-rw-r--r--. 1 atguigu atguigu  399 Jun 15 14:53 id_rsa.pub
-rw-r--r--. 1 atguigu atguigu  370 Jun 15 12:58 known_hosts
[atguigu@hadoop322-node01 hadoop-3.2.2]$ 

# 将hadoop322-node01的公钥拷贝到hadoop322-node02上
[atguigu@hadoop322-node01 hadoop-3.2.2]$ ssh-copy-id hadoop322-node02
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/atguigu/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
atguigu@hadoop322-node02's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'hadoop322-node02'"
and check to make sure that only the key(s) you wanted were added.

# 在hadoop322-node02上查看是否拷贝成功了
[atguigu@hadoop103 hadoop-3.2.2]$ ll ~/.ssh/
total 16
-rw-------. 1 atguigu atguigu  399 Jun 15 14:57 authorized_keys
-rw-------. 1 atguigu atguigu 1679 Jun 15 14:53 id_rsa
-rw-r--r--. 1 atguigu atguigu  399 Jun 15 14:53 id_rsa.pub
-rw-r--r--. 1 atguigu atguigu  185 Jun 15 12:52 known_hosts

# 验证从hadoop322-node01是否能免密SSH到hadoop322-node02
[atguigu@hadoop102 hadoop-3.2.2]$ ssh hadoop322-node02
```

同理对hadoop322-node01配置可免密登录hadoop322-node03。

<font color="red">**注意：</br>还需要在hadoop322-node01、hadoop322-node02、hadoop322-node03都要配置root用户、atguigu用户的免密ssh。配置方式如上所示。**</font>

### 2.3.4 安装和配置JDK

#### 2.3.4.1 卸载openJDK

VM ware中自带了openjdk，其中openJDK安装好的目录位于`/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-1.el7.x86_64`

那么应该如何配置`JAVA_HOME`等环境变量呢？

首先，咱删除默认安装的openJDK，然后手动安装指定版本的JDK，再配置环境变量即可。

[使用CentOS7卸载自带jdk安装自己的JDK1.8](https://blog.csdn.net/hui_2016/article/details/69941850?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-7.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-7.control)

```shell
[root@hadoop101 alternatives]$  rpm -qa | grep java
javapackages-tools-3.4.1-11.el7.noarch
java-1.7.0-openjdk-headless-1.7.0.251-2.6.21.1.el7.x86_64
python-javapackages-3.4.1-11.el7.noarch
tzdata-java-2019c-1.el7.noarch
java-1.8.0-openjdk-1.8.0.242.b08-1.el7.x86_64
java-1.8.0-openjdk-headless-1.8.0.242.b08-1.el7.x86_64
java-1.7.0-openjdk-1.7.0.251-2.6.21.1.el7.x86_64

[root@hadoop101 opt]# rpm -e --nodeps java-1.7.0-openjdk-headless-1.7.0.251-2.6.21.1.el7.x86_64
[root@hadoop101 opt]# rpm -e --nodeps java-1.8.0-openjdk-1.8.0.242.b08-1.el7.x86_64
[root@hadoop101 opt]# rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.242.b08-1.el7.x86_64
[root@hadoop101 opt]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.251-2.6.21.1.el7.x86_64
# 相比于之前，少了4个openjdk的包了
[root@hadoop101 opt]# rpm -qa | grep java
javapackages-tools-3.4.1-11.el7.noarch
python-javapackages-3.4.1-11.el7.noarch
tzdata-java-2019c-1.el7.noarch

```

#### 2.3.4.2 安装JDK8

官网下载所需版本的JDK，然后拷贝到虚拟机中的`/opt/software`目录

```shell
[atguigu@hadoop101 hadoop]$ cd /opt/softwares
[atguigu@hadoop101 ~]$ tar -zxvf jdk-8u291-linux-x64.tar.gz -C /opt/module
```

####  2.3.4.3 配置JAVA_HOME等环境变量

```bash
[atguigu@hadoop101 software]$ sudo vim /etc/profile.d/hadoop322_env.sh
# 修改环境变量
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_291
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
[atguigu@hadoop101 ~]$ source /etc/profile
[atguigu@hadoop101 ~]$ java -version
[atguigu@hadoop101 ~]$ javac --help
```

**为什么在`/etc/profile.d`而不是在`/etc/profile`文件中直接添加/修改环境变量呢？**

> 在 /etc/profile 这个文件中有这么一段 shell, 会在每次启动时自动加载 profile.d 下的每个配置
>
> ```bash
> if [ -d /etc/profile.d ]; then
>   for i in /etc/profile.d/*.sh; do
>     if [ -r $i ]; then
>       . $i
>     fi
>   done
>   unset i
> fi
> ```
>
> 区别
> 1、都用来设置环境变量文件
>
> 2、/etc/profile.d/ 高度解耦, 比 /etc/profile 好维护，不想要什么变量直接删除 /etc/profile.d/ 下对应的 shell 脚本即可
>
> 3、/etc/profile 和 /etc/profile.d 同样是登录（login）级别的变量，当用户重新登录 shell 时会触发。所以效果一致。
>
> 4、设置登录级别的变量，重新登录 shell，或者 source /etc/profile。

### 2.3.5 安装和配置Hadoop3.2.2

提前将hadoop3.2.2的tar包`hadoop-3.2.2.tar.gz`放到`/opt/softwares`目录下。

#### 2.3.5.1 安装Hadoop3.2.2

```bash
[atguigu@hadoop101 hadoop]$ cd /opt/softwares
[atguigu@hadoop101 software]$ tar -zxvf hadoop-3.2.2.tar.gz -C /opt/module
```

####  2.3.5.2 配置HADOOP_HOME等环境变量

```bash
[atguigu@hadoop101 software]$ sudo vim /etc/profile.d/hadoop322_env.sh
#HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.2.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

[atguigu@hadoop101 software]$ source /etc/profile
[atguigu@hadoop101 software]$ hadoop --help
```

### 2.3.6 调整集群配置

core-site.xml

```xml
<configuration>
	<property>
  	<name>fs.defaultFS</name>
    <value>hdfs://hadoop322-node01:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/modules/hadoop-3.2.2/data/tmp</value>
  </property>
</configuration>
```

hdfs-site.xml

```xml
<configuration>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop322-node03:50090</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>${hadoop.tmp.dir}/dfs/name</value>
  </property>
  <property>
    <name>dfs.blocksize</name>
    <value>134217728</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>${hadoop.tmp.dir}/dfs/data</value>
  </property>
</configuration>
```

yarn-site.xml

```xml
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop322-node02</value>
  </property>
  <property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.env-whitelist</name>
<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value>
  </property>
</configuration>
```

mapred-site.xml

```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  
  <!-- 历史服务器端地址 -->
  <property>
    <name>mapreduce.jobhistory.address</name>
      <value>hadoop101:10020</value>
  </property>
  <!-- 历史服务器web端地址 -->
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
      <value>hadoop101:19888</value>
	</property>
</configuration>
```

etc/hadoop/workers

```shell
hadoop322-node01
hadoop322-node02
hadoop322-node03
```

### 2.3.7 启动Hadoop集群服务

#### 2.3.7.1 逐个启动

1. 格式化`NameNode`（**第一次启动时格式化，以后就不要总格式化**）

   ```shell
   bin/hdfs namenode -format
   ```

2. 启动NameNode

   ```shell
   sbin/hadoop-daemon.sh start namenode
   # hadoop3建议使用如下命令启动
   hdfs --daemon start namenode
   ```

3. 启动NameNode

   ```shell
   sbin/hadoop-daemon.sh start datanode
   # hadoop3建议使用如下命令启动
   hdfs --daemon start datanode
   ```

4. 启动ResourceManager

   ```bash
   sbin/yarn-daemon.sh start resourcemanager
   # hadoop3建议使用如下命令启动
   yarn --daemon start resourcemanager
   ```

   

5. 启动NodeManager

   ```bash
   sbin/yarn-daemon.sh start nodemanager
   # hadoop3建议使用如下命令启动
   yarn --daemon start nodemanager
   ```

   

6. 启动MapReduce历史服务器

   ```bash
   sbin/mr-jobhistory-daemon.sh start historyserver
   # hadoop3建议使用如下命令启动
   mapred --daemon start historyserver
   ```

   

#### 2.3.7.2 群起

```bash
#启动YARN的服务
sbin/start-dfs.sh

#启动YARN的服务
sbin/start-yarn.sh
```





MapReduce的JobHistoryServer服务。

![image-20230213161316608](Hadoop3.x.assets/image-20230213161316608.png)

如下图所示：

![image-20230213161341891](Hadoop3.x.assets/image-20230213161341891.png)

## 2.4 HDFS HA 非联邦完全分布式

有两种HA的方案：

- `NameNode HA with QJM`。使用`the Quorum Journal Manager (QJM)`在`Active NameNode`和`Standby NameNode`之前共享edit logs。
- `NameNode HA with NFS`。使用`NFS`在`Active NameNode`和`Standby NameNode`之前共享edit logs。

### 2.4.1 `NameNode HA with QJM`

参考官方文档：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html

让我们回到配置`core-site.xml`, `hdfs-site.xml`, `yarn-site.xml`, `mapred-site.xml`之前。

在`Hdoop2.0.0`之前，NameNode一直都是单点故障的（single point of failure (SPOF)），只要NameNode所在机器或进程不可用，则整个集群的HDFS就不可用了。

- 机器挂了，或者NameNode进程挂了，则直到机器/进程重启之，集群才能变为可用。
- 集群/软件升级，会导致一定时间窗口内集群不可用。

### 2.4.2 HA的设计架构

在典型的HA集群中，两个或多个单独的机器被配置为NameNodes。在任何时间点，只有一个NameNode处于active状态，而其他NameNode处于standby状态。Active NameNode负责集群中的所有客户端操作，而Standby只是在必要时保持足够的状态以提供快速的故障恢复（failover）。

**为了实现active NameNode和standby NameNode的状态保持一致**，引入了"JournalNodes"来实现二者之前的通信。`active NameNode`将namespace的改动记录到**大多数的JN**中。然后`standby NameNode`从JN中获取namespace的的改动，并不断监视edit log的变动，将edit log应用到自身内存中的namespace。在发生故障转移时，备用服务器将确保在将自身提升到活动状态之前，已从`JournalNode`中读取所有编辑。这确保在发生故障切换之前，命名空间状态完全同步。

**为了提供快速故障切换，备用节点还必须具有关于集群中块位置的最新信息。**为了实现这一点，DataNode配置了所有NameNode的位置，并向所有NameNode发送块位置信息和心跳。

如果同一时刻有多个`active NameNode`，则会发生"split-brain scenario"（脑裂），会导致数据丢失或其他问题。同一时刻只有一个`active NameNode`保证了，同一时刻只有一个 NameNode可以往`Journal Nodes`中写入edit log。



| hadoop322-noe01                                              | hadoop322-node02                                             | hadoop322-node03                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| NameNode<br>DataNode<br>NodeManager<br>JournalNode<br>ZKFC<br>JobHistoryServer | NameNode<br>ResourceManager<br>DataNode<br>JournalNode<br>ZKFC<br/>NodeManager | NameNode<br>DataNode<br>JournalNode<br>ZKFC<br/>NodeManager |



### 2.4.3 硬件准备

- 建议NameNode节点的硬件配置相同。
- `JournalNode`守护进程是比较轻量的，可以放置在其它守护进程（如NameNode、JobTracker、ResourceManager等）如所在节点。但是，就公有云的EMR（MRS）之类的paas产品而言，都是将JournalNode和ZooKeeper放置在相同的节点，一般为3个节点，配置一般2C4G 100GB就可以了。

**请注意，在HA集群中，Standby NameNode还执行命名空间状态的检查点，因此不需要在HA集群内运行Secondary NameNode、Checkpoint Node或BackupNode。事实上，这样做是错误的。这也允许正在重新配置非启用HA的HDFS群集以启用HA的用户重新使用以前专用于Secondary NameNode的硬件。？没看懂TODO**

### 2.4.4 部署

与Federation配置类似，HA配置是向后兼容的，并允许现有的单个NameNode配置在没有更改的情况下工作。新配置被设计为使得集群中的所有节点可以具有相同的配置，而不需要基于节点的类型将不同的配置文件部署到不同的机器。?没看懂TODO

#### 2.4.4.1 配置HDFS HA集群

与`HDFS Federation`一样，HA集群重用`nameservice ID`来标识单个HDFS实例，该实例实际上可能由多个HA NameNode组成。此外，HA还添加了名为`NameNode ID`的新抽象。集群中每个不同的NameNode都有一个不同的`NameNode ID`来区分它。为了支持所有NameNode的单个配置文件，相关的配置参数都以`nameservice ID`和`NameNode ID`作为后缀。

##### 配置`core-site.xml`

```xml
<configuration>
  <!-- 默认FS的路径前缀 -->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://mycluster</value>
	</property>
</configuration>
```

- **fs.defaultFS**: Hadoop FS客户端在没有指明FS前缀时，使用的默认路径前缀。

##### 配置`hdfs-site.xml`



```xml
<configuration>
    <!-- nameservice的名称，若为联邦，此处为逗号分隔的nameservice的列表 -->
    <property>
      <name>dfs.nameservices</name>
      <value>mycluster</value>
    </property>
    
    <!-- 在该nameservice中NameNode的id，此处为逗号分隔的NameNode ID列表。
		最少2个，最多5个，建议3个，因为过多的话，NameNode之前的通信开销过大。 -->
    <property>
  	  <name>dfs.ha.namenodes.mycluster</name>
      <value>nn1,nn2,nn3</value>
	</property>
    
    <!-- 每个NameNode监听的RPC地址 -->
    <property>
      <name>dfs.namenode.rpc-address.mycluster.nn1</name>
  	  <value>hadoop322-node01:8020</value>
	</property>
	<property>
      <name>dfs.namenode.rpc-address.mycluster.nn2</name>
      <value>hadoop322-node02:8020</value>
	</property>
	<property>
  	  <name>dfs.namenode.rpc-address.mycluster.nn3</name>
      <value>hadoop322-node03:8020</value>
	</property>
    
    <!-- 每个NameNode监听的HTTP地址，如果启用security特性，则需配置dfs.namenode.https-address.mycluster.? -->
    <property>
      <name>dfs.namenode.http-address.mycluster.nn1</name>
      <value>hadoop322-node01:9870</value>
    </property>
    <property>
      <name>dfs.namenode.http-address.mycluster.nn2</name>
      <value>hadoop322-node02:9870</value>
	</property>
	<property>
  	  <name>dfs.namenode.http-address.mycluster.nn3</name>
      <value>hadoop322-node03:9870</value>
    </property>
    
    <!-- NameNode向这些JN写/读edits log，URI地址 -->
    <property>
      <name>dfs.namenode.shared.edits.dir</name>
      <value>qjournal://hadoop322-node01:8485;hadoop322-node02:8485;hadoop322-node03:8485/mycluster</value>
	</property>
    
    <!-- HDFS客户端用于联系ActiveNameNode的Java类 -->
    <property>
  	  <name>dfs.client.failover.proxy.provider.mycluster</name>
  	  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
  
    <!-- 故障转移期间用来fence Active NameNode -->
    <property>
      <name>dfs.ha.fencing.methods</name>
      <value>sshfence</value>
    </property>
    <property>
      <name>dfs.ha.fencing.ssh.private-key-files</name>
      <value>/home/hadoop/.ssh/id_rsa</value>
    </property>
   
  <!-- Journal Nodes在本地存储edits文件的路径 -->
  <property>
    <name>dfs.journalnode.edits.dir</name>
    <value>/opt/modules/hadoop-3.2.2/data/journalnode/</value>
	</property>
  
  <!-- 是否阻止安全模式中的namenodes变为Active。-->
  <property>
    <name>dfs.ha.nn.not-become-active-in-safemode</name>
    <value>true</value>
	</property>
</configuration>
```

- **dfs.nameservices**

  `nameservice`的名字，如mycluster。若为联邦，此处为逗号分隔的nameservice的列表

  后续的配置中会用到此配置项的值。

- **dfs.ha.namenodes.[nameservice ID]**

  在指定`nameservice`中每个`NameNode`的唯一标识符。为逗号分隔的`NameNodes`的ID列表。通常为`nn1,nn2,nn3`，以此类推。

  **最少2个，最多5个，建议3个，因为过多的话，`NameNode`之前的通信开销过大**。

- **dfs.namenode.rpc-address.[nameservice ID].[name node ID]**

  每个NameNode监听的RPC地址。`host:port`格式，host为NameName进程所在主机，port为NameNode的`IPC`端口。

- **dfs.namenode.http-address.[nameservice ID].[name node ID]**

  每个NameNode监听的HTTP地址，如果启用security特性，则需配置`dfs.namenode.https-address.[nameservice ID].[name node ID]`

- **dfs.namenode.shared.edits.dir**

  `NameNode`读/写edits log的`Journal Nodes`的URI，这些`Journal Nodes`提供了`edits log`的共享存储。`Active NameNode`向`Journal Nodes`写入指定`nameserviceId`的edits log，其余`standby NameNode`向`Journal Nodes`读取指定`nameserviceId`的edits log。**至少需要配置一个Journal Node的URI**。

  格式为：`qjournal://host1:port1;host2:port2;host3:port3/journalId`，通常情况下，`journalId`与`nameserviceId`保持一致。联邦集群中可以存在多个`nameserviceId`，对应也有多个`journalId`，他们可以使用同一组`Journal Nodes`，通过不同的`journalId`来区分就可以了。

- **dfs.client.failover.proxy.provider.[nameservice ID]** 

  HDFS客户端用于联系ActiveNameNode的Java类。

  此类决定了这些NameNodes中，哪个是Active NameNode，此NameNode负责处理客户端的请求。

  Hadoop官方提供了两个类：**ConfiguredFailoverProxyProvider** 和 **RequestHedgingProxyProvider**，也可以自定义。

- **dfs.ha.fencing.methods**

  在故障切换期间用于**隔离**Active NameNode的脚本或Java类的列表。

  - **sshfence** 。ssh到Active NameNode所在机器并kill进程。

    SSH连接到目标节点，并使用fuser终止监听TCP端口的进程。为了使此隔离选项发挥作用，它必须能够在不提供密码的情况下SSH到目标节点。因此，还必须配置dfs.ha.fening.sh.private-key-files选项，这是一个逗号分隔的ssh私钥文件列表。

    还可以配置指定ssh时使用的用户名和端口，也可以指定ssh的超时时间。

  - **shell**。运行shell命令来fence Active NameNode。

- **dfs.journalnode.edits.dir**

  Journal Nodes在本地存储edits文件的路径。默认值为`/tmp/hadoop/dfs/journalnode/`。

- **dfs.ha.nn.not-become-active-in-safemode**

  是否阻止安全模式中的namenodes变为Active。

#### 2.4.4.2 启动HDFS HA集群

1、在所有`Journal Node`启动`journalnode`服务

```bash
hdfs --daemon start journalnode
```

注意：启动JournalNodes后，必须首先同步NameNodes上的磁盘元数据 。第二步就是做这个事情的。

2、`格式化NameNode`或`初始化JournalNode`

有两种情况：

- 创建一个新的HDFS集群。

  在某一个NameNode上执行`hdfs namenode -format`格式化集群，只在第一次启动HDFS HA集群时使用。

- 将非HA集群的`NameNode`转换为HA。

  运行命令`hdfs namenode -ininitializeSharedEdits`，该命令将使用NameNode本地的edits目录中的数据来初始化`JournalNode`。

根据需求，在nn1上选择使用`hdfs namenode -format`还是`hdfs namenode -initializeSharedEdits`

```bash
hdfs namenode -format
或
hdfs namenode -initializeSharedEdits
```

3、启动nn1的`NameNode`

在nn1执行如下命令：

```bash
hdfs --daemon start namenode
```

旧的命令是`hadoop-daemon.sh start namenode`，现在建议使用上面的命令了。

4、在nn1启动了`NameNode`之后。将nn1的元数据信息（`{dfs.namenode.name.dir}` 目录的内容）复制到nn2、nn3的`{dfs.namenode.name.dir}` 目录下(**只在第一次启动HDFS HA集群时使用**)

分别在nn2、nn3执行如下命令：

```bash
hdfs namenode -bootstrapStandby
```

![image-20230214142714673](Hadoop3.x.assets/image-20230214142714673.png)

输出如上信息，表示已经做好了将NameNode设置为standby的工作了。可以启动standby NameNode了

5、启动nn2和nn3的`NameNode`

```bash
hdfs --daemon start namenode
```

3个NameNode都启动后，发现均为`standby`。

nn1

![image-20230214144231824](Hadoop3.x.assets/image-20230214144231824.png)

nn2

![image-20230214144334363](Hadoop3.x.assets/image-20230214144334363.png)

nn3

![image-20230214144301091](Hadoop3.x.assets/image-20230214144301091.png)

<font color=red>**注意：的是Stanby状态的NameNode是不能对外提供服务的！**</font>

![image-20230214143230955](Hadoop3.x.assets/image-20230214143230955.png)



命令行查看也是如此：

```bash
hdfs haadmin -getAllServiceState
```

![image-20230214144600294](Hadoop3.x.assets/image-20230214144600294.png)



6、启动所有`DataNode`

在nn1, nn2, nn3启动`DataNode`服务。

```bash
hdfs --daemon start datanode
```

jps查看是否进程都在，HDFS Web UI查看 live Nodes 是否为3。

7、将nn1切换为`Active NameNode`

```bash
hdfs haadmin -ns mycluster -transitionToActive nn1
或者
hdfs haadmin -transitionToActive nn1
```

![image-20230214152737107](Hadoop3.x.assets/image-20230214152737107.png)

<font color=red>**注意：如果nn1进程挂掉了，使用`bin/hdfs haadmin -transitionToActive nn2`并不能让nn2成为Active。**</font>why?

![image-20230214154332460](Hadoop3.x.assets/image-20230214154332460.png)

**使用`hdfs haadmin -failover nn1 nn2`也无法实现故障转移。**

![image-20230214154738405](Hadoop3.x.assets/image-20230214154738405.png)

<font color=red>**注意：HDFS HA 想要手动故障转移，则nn1和nn2必须都是正常启动的，然后使用如下命令进行手动故障转移**</font>。

```bash
hdfs haadmin -transitionToActive nn2
```



#### 2.4.4.3 自动故障转移配置

自动故障转移依赖`ZooKeeper`和`ZKFailoverController`（缩写为ZKFC）。

ZooKeeper主要实现两个作用：**故障检测**和**Active NameNode选举**。

- **故障检测**：集群中的每个NameNode机器都在ZooKeeper中维护一个持久会话。如果机器崩溃，ZooKeeper会话将过期，通知其他NameNode应该触发故障转移。
- **Active NameNode选举**：ZooKeeper提供了一种简单的机制，以独占方式将节点选为活动节点。如果当前活动的NameNode崩溃，另一个节点可能会在ZooKeeper中获取一个特殊的独占锁，指示它应该成为下一个活动节点。

**ZKFC**既是一个ZooKeeper client，又监控和管理NameNode的状态。每一个NameNode所在主机都会运行一个**ZKFC**。

**ZKFC**有三个职责：

- 健康监测（**Health monitor**）

  ZKFC使用健康检查命令定期ping其本地NameNode。只要NameNode以健康状态及时响应，ZKFC就会认为该节点是健康的。如果节点已崩溃、僵死或以其他方式进入不正常状态，health monitor会将其标记为不正常状态。

- **ZooKeeper session management**

  当本地NameNode运行正常时，ZKFC会在ZooKeeper中持有一个会话。如果本地NameNode处于Active状态，它还持有一个特殊的“锁”znode。该锁使用ZooKeeper对“短暂”节点的支持；如果会话过期，锁定节点将被自动删除。

- **ZooKeeper-based election**

  如果本地NameNode运行正常，并且ZKFC发现当前没有其他节点持有锁znode，则它自己将尝试获取锁。如果它成功了，那么它就“赢得了选举”，并负责运行故障切换以使其本地NameNode处于Active状态。故障切换过程类似于上述手动故障切换：首先，如果需要，前一个活动状态被隔离，然后本地NameNode转换为Active状态。

##### 2.4.4.3.1 部署ZooKeeper

###### 下载ZooKeeper

官方下载地址：https://archive.apache.org/dist/zookeeper/

###### 解压ZooKeeper

```bash
tar -zxvf apache-zookeeper-3.6.1-bin.tar.gz -C /opt/modules/
```

###### 配置ZooKeeper

【可选】安装`ansible`(操作系统为centos7.x)

```bash
yum install epel-release
yum install ansible
```

在[nn1]创建`ansible`使用的主机名文件

```bash
vim ~/ansible_workers
[workers]
hadoop322-node01
hadoop322-node02
hadoop322-node03
```

创建ZooKeeper的数据目录

​	没有安装ansible的话，就每个节点上去单独创建此目录即可。

```bash
ansible workers -i ansible_hosts -m shell -a 'mkdir /opt/modules/apache-zookeeper-3.6.1-bin/zkData'
```

为每个ZooKeeper节点创建`myid`文件

```bash
# [nn1]机器上
echo 1 > /opt/modules/apache-zookeeper-3.6.1-bin/zkData/myid
# [nn2]机器上
echo 2 > /opt/modules/apache-zookeeper-3.6.1-bin/zkData/myid
# [nn3]机器上
echo 3 > /opt/modules/apache-zookeeper-3.6.1-bin/zkData/myid
```

修改`dataDir=/opt/modules/apache-zookeeper-3.6.1-bin/zkData`

```shell
[root@agent apache-zookeeper-3.5.9-bin]# cp conf/zoo_sample.cfg conf/zoo.cfg
[root@agent apache-zookeeper-3.5.9-bin]# vim conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
# 修改ZooKeeper的数据存储目录
dataDir=/opt/modules/apache-zookeeper-3.6.1-bin/zkData
clientPort=2181
# 增加如下内容
server.1=hadoop01:2888:3888
server.2=hadoop01:2888:3888
server.3=hadoop01:2888:3888
```

###### 启动zk集群

```shel
# 第一台机器
[root@agent apache-zookeeper-3.5.9-bin]# bin/zkServer.sh start
# 第二台机器
[root@agent apache-zookeeper-3.5.9-bin]# bin/zkServer.sh start
# 第三台机器
[root@agent apache-zookeeper-3.5.9-bin]# bin/zkServer.sh start
```

也可以通过如下命令启动每个ZooKeeper服务器进程：

```bash
java -cp zookeeper.jar:lib/*:conf org.apache.zookeeper.server.quorum.QuorumPeerMain zoo.conf
```

***注意：需要修改其中的zoo.conf为实际的ZooKeeper的配置文件。***

**官方建议ZooKeeper的JVM堆内存为3 ~ 4G，避免影响ZooKeeper的性能。以我在公有云中的经验（腾讯云EMR），一般选择4C2G 100GB磁盘就够了。**

如果要修改ZooKeeper进程的JVM内存大小，请参考：https://www.cnblogs.com/LiuChang-blog/p/15127157.html

##### 2.4.4.3.2 配置自动故障转移

***在配置HDFS HA集群自动故障转移之前，需要先关闭HA集群。因为在集群运行时无法从手动故障转移切换至自动故障转移。***

`core-site.xml`中增加如下配置项，配置的是ZooKeeper的地址，根据实际情况修改为自己的ZooKeeper集群。

```xml
 <property>
   <name>ha.zookeeper.quorum</name>
   <value>hadoop322-node01:2181,hadoop322-node02:2181,hadoop322-node03:2181</value>
 </property>
```



`hdfs-site.xml`中增加

```xml
 <property>
   <name>dfs.ha.automatic-failover.enabled</name>
   <value>true</value>
 </property>
```

<font color="red">***如果是联邦集群，则可以通过`dfs.ha.automatic-failover.enabled.[my-nameservice-ID]`来配置每个`nameservice`的配置。***</font>

初始化HDFS HA在ZooKeeper中的状态（在某个NameNode节点执行）

```bash
$HADOOP_HOME/bin/hdfs zkfc -formatZK
```

​	**这会在ZooKeeper中创建HDFS HA集群所需的znode，一般为[hadoop-ha]**。

启动HDFS HA集群中的所有服务进程。

```bash
$HADOOP_HOME/sbin/start-dfs.sh
```

手动启动`ZKFC`

```bash
$HADOOP_HOME/bin/hdfs --daemon start zkfc
```

当所有进程都正常启动后，NameNodes们会通过在ZooKeeper中抢占x znode，来进行选举，确认哪个NameNode为Active。会多出来两个znode：`ActiveBreadCrumb`和`ActiveStandbyElectorLock`

![image-20230222110503331](Hadoop3.x.assets/image-20230222110503331.png)

![image-20230222110834921](Hadoop3.x.assets/image-20230222110834921.png)

从上图可以看出，当前的Active NameNode是 hadoop322-node01。

**Tip**s：关于自动故障转移的测试见博客：https://blog.csdn.net/m0_37613244/article/details/114504071

#### 2.4.4.4 `In-Progress Edit Log Tailing`

在默认设置下，`Standby NameNode`只会应用已完成的edits log，`edits_inprogress_0000000000000000322`之类的未完成的edits log，`Standby NameNode`并不会应用到自身的namespace中。

如果希望有一个具有最新namespace的`Standby NameNode`，则可以启用`In-Progress Edit Log Tailing`。启用之后，`Standby NameNode`将尝试从`JournalNode`的内存缓存中获取edits。

- **dfs.ha.tail-edits.in-progress**

  是否对正在进行的edits log启用跟踪。启用后，在`Journal Node`将会在内存中缓存in-progress的edits log。默认情况下禁用

- **dfs.journalnode.edit-cache-size.bytes**

  JournalNode上edits的内存缓存的大小。一般情况，每个edit大约需要200字节，因此默认值1048576（1MB）可以容纳大约5000个edit事务。根据实际情况调整。

#### 2.4.4.5 HDFS HA集群支持`Load Balancer`

在LB的健康检测中，使用[http://NN_HOSTNAME/isActive](http://nn_hostname/isActive)，来检测哪个NN是Acitve的。Active NN会返回200，Standby NN返回405。

#### 2.4.4.6 HDFS HA集群的Upgrade/Finalization/Rollback

参考：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html

## 2.5 HA 联邦完全分布式

参考官方文档：https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithNFS.html

联邦模式也有HA和非HA的区别，见博客：https://blog.csdn.net/weixin_34244102/article/details/91645766

https://my.oschina.net/bochs/blog/789612

TODO

## 2.6 YARN的高可用

官方文档：https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/ResourceManagerHA.html

在hadoop2.4之前，YARN集群中的`ResourceManager`一直是存在单点故障的。

YARN HA的架构图如下：

![Overview of ResourceManager High Availability](Hadoop3.x.assets/rm-ha-overview.png)

### 2.6.1 RM故障转移

RM的HA也是依赖ZooKeeper的，也是Active/Standby的。

#### 2.6.1.1 手动故障转移

如果没有开启RM的自动故障转移，我们也可以通过手动故障转移。

TODO

```bash
yarn rmadin 
```



#### 2.6.1.2 自动故障转移

RM中可以内嵌`ActiveStandbyElector`，用来做**故障检测**和**leader选举**。因此YARN中不需要类似ZKFC这种独立的服务进程。

#### 2.6.1.3 配置YARN

`yarn-site.xml`

```xml
<!-- 启用YARN resourcemanager的HA -->
<property>
  <name>yarn.resourcemanager.ha.enabled</name>
  <value>true</value>
</property>
<!-- YARN集群id -->
<property>
  <name>yarn.resourcemanager.cluster-id</name>
  <value>cluster-yarn1</value>
</property>
<!-- YARN HA集群中的RM的id -->
<property>
	<name>yarn.resourcemanager.ha.rm-ids</name>
  <value>rm1,rm2</value>
</property>
<!-- YARN集群中RM的地址 -->
<property>
	<name>yarn.resourcemanager.hostname.rm1</name>
  <value>hadoop322-node01</value>
</property>
<property>
	<name>yarn.resourcemanager.hostname.rm2</name>
  <value>hadoop322-node03</value>
</property>

<!-- 在YARN HA模式下，下面这两个配置必须显式配置，否则会连不上webapp -->
<property>
	<name>yarn.resourcemanager.webapp.address.rm1</name>
  <value>hadoop322-node01:8088</value>
</property>
<property>
	<name>yarn.resourcemanager.webapp.address.rm2</name>
  <value>hadoop322-node03:8088</value>
</property>

<!-- 启动RM后，允许其恢复状态。开启后必须指定yarn.resourcemanager.store.class -->
<property>
  <name>yarn.resourcemanager.recovery.enabled</name>
  <value>true</value>
</property>
<!-- 持久化RM状态信息的类 -->
<property>
  <name>yarn.resourcemanager.store.class</name>
  <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
</property>

<!-- ZooKeeper集群地址 -->
<property>
  <name>hadoop.zk.address</name>
  <value>hadoop322-node01:2181,hadoop322-node02:2181,hadoop322-node03:2181</value>
</property>
```

#### 2.6.1.4 启动YARN

```bash
sbin/start-yarn.sh
```

连接ZooKeeper查看发现多了两个znode：`rmstore`和`yarn-leader-election`。一个是用来**存储状态信息的**，一个是用来做**leader选举的**。

![image-20230222170233962](Hadoop3.x.assets/image-20230222170233962.png)

如下所示，表示当前的Active RM 是rm2。

![image-20230222170205921](Hadoop3.x.assets/image-20230222170205921.png)

<font color="red">**注意：**</font>

<font color="red">**YARN把RM的状态信息存储在本地文件系统或ZooKeeper集群中，当RM重启后可以依据这些数据恢复状态。**</font>

<font color="red">**当`Standby RM` 运行时，访问`Standby RM`的`web ui`的请求会重定向到`Active RM`。**</font>

<font color="red">**当`Standby RM` 运行时，访问`Standby RM`的`REST API`请求会重定向到`Active RM`。**</font>

### 2.6.2 YARN 命令

查看RM的状态。

```bash
 $ yarn rmadmin -getServiceState rm1
 active
 
 $ yarn rmadmin -getServiceState rm2
 standby
```

如果启用了自动故障转移，则不能使用`-transitionTOStandby`，如果一定要使用需要加`-forcemanual`。

```bash
 $ yarn rmadmin -transitionToStandby rm1
 Automatic failover is enabled for org.apache.hadoop.yarn.client.RMHAServiceTarget@1d8299fd
 Refusing to manually manage HA state, since it may cause
 a split-brain scenario or other incorrect state.
 If you are very sure you know what you are doing, please
 specify the forcemanual flag.
 
  $ yarn rmadmin -transitionToStandby -forcemanual rm1
```

其它命令见：https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/YarnCommands.html

### 2.6.3 关于RM重启

参考官方文档：https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/ResourceManagerRestart.html

RM有两种重启类型：

- `Non-work-preserving RM restart`：

  RM将在客户端提交application时将application元数据存储在state-store（按照上述配置的话，就是`ZooKeeper`集群）。那么当RM重新启动时，它就可以从状态存储中获取application元数据并重新运行已提交的application。如果在RM停机之前申请已经完成（即失败、终止或完成），RM将不会重新提交申请。用户无需重新提交application。

- `Work-preserving RM restart`：

  在重启时，结合NodeManager反映的contianer状态和ApplicationMaster反映的container请求来重新构建RM的状态。这种方式不会重新运行之前正在运行的application。

`yarn-site.xml`中的`yarn.resourcemanager.store.class`支持三种状态存储的方式：

- `org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore`

  使用ZooKeeper做状态存储，**支持fence机制。**

- `org.apache.hadoop.yarn.server.resourcemanager.recovery.FileSystemRMStateStore`

  默认。使用本地文件系统做状态存储，**不支持fence机制。**

- `org.apache.hadoop.yarn.server.resourcemanager.recovery.LeveldbRMStateStore`

  **不支持fence机制。**

### 2.6.4 配置`Load Balancer`

YARN HA集群支持配置LB，通过RM web ui 的`/isAcitve` HTTP请求，可以对RM服务做健康检查。Active RM会返回200状态码，而其它会返回405状态码。

# 3. Hadoop命令

## 管理员命令

```bash
hdfs haadmin --help
Usage: haadmin
    [-transitionToActive <serviceId>]
    [-transitionToStandby <serviceId>]
    [-failover [--forcefence] [--forceactive] <serviceId> <serviceId>]
    [-getServiceState <serviceId>]
    [-getAllServiceState]
    [-checkHealth <serviceId>]
    [-help <command>]
```

- **-transitionToActive**：将指定NameNode的状态转换为Active。
- **-transitionToStandby** ：将指定NameNode的状态转换为Standby。
- **-failover** ：进行故障转移，在将Active从第一个NameNode转移到第二个NameNode。
- **-getServiceState** ：获取指定NameNode的状态，Active/Standby。
- **-getAllServiceState** ：获取**所有**NameNode的状态，Active/Standby。
- **-checkHealth** ：获取指定NameNode的健康状态。健康则什么结果都没有，不健康则会有具体的报错信息。**功能目前尚未完全实现，不建议使用。**

<font color="red">**注意：不执行fence，因此一般不建议使用这两个命令，而是建议使用`hdfs haadmin -failover`。**</font>

