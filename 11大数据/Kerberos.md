# 1. Kerberos部署

HDFS中，启动HDFS进程的用户是谁，谁就是HDFS的超级用户（superuser）。

![image-20211011213554700](D:/work/myGitRepositories/study-notes/11大数据/HDFS.assets/image-20211011213554700.png)

在之前的介绍中，并不能保证数据的安全，因为只要能访问HDFS的用户，都能对HDFS中的所有数据进行读写操作。这样数据非常不安全。

要保证HDFS的安全，需要保证两个部分：

- **认证（authentication ）**，验证用户的身份。HDFS中，认证主要通过`kerberos`来实现。
- **授权（Authorization）**，用户是否有对应操作的权限。Hadoop集群的授权主要通过`Sentry`来实现，现在更多地是用`Ranger`了。

## 1.1 Kerberos概述

### 1.1.1 什么是Kerberos

Kerberos是一种计算机网络认证协议，用来在非安全网络中，对个人通信以安全的手段进行==身份认证==。这个词又指麻省理工学院为这个协议开发的一套计算机软件。软件设计上采用客户端/服务器结构，并且能够进行相互认证，即客户端和服务端均可对对方进行身份认证，可以用于防止窃听、防止重放攻击、保护数据完整性等场合，是一种应用对称密钥体制进行密钥管理的系统。

### 1.1.2 Kerberos术语

1. KDC（Key Distribute Center）：密钥分发中心，负责存储用户信息，管理发放票据。
2. Realm（域）：Kerberos所管理的一个领域或范围，称之为一个Realm。
3. Principal（主体）：Kerberos所管理的一个用户或者一个服务，可以理解为Kerberos中保存的一个账号，其格式通常如下：`primary/instance@realm名`
4. keytab：Kerberos中的用户认证，可通过密码或者密钥文件证明省份，keytab指密钥文件。

### 1.1.3 Kerberos认证原理

![image-20211011220011686](Kerberos.assets/image-20211011220011686.png "Kerberos认证原理")

## 1.2 Kerberos安装

### 1.2.1 安装Kerberos相关服务

选择集群中的一台主机（hadoop102）作为Kerberos服务端，安装KDC，所有主机都需要部署Kerberos客户端。

服务端主机执行以下安装命令

```shell
[root@hadoop102 ~]# yum install -y krb5-server
```

客户端主机执行以下安装命令：

```shell
[root@hadoop102 ~]# yum install -y krb5-workstation krb5-libs
[root@hadoop103 ~]# yum install -y krb5-workstation krb5-libs
[root@hadoop104 ~]# yum install -y krb5-workstation krb5-libs
```



### 1.2.2 修改配置文件

1. 服务端主机（hadoop102）

   修改`/var/kerberso/krb5kdc/kdc.conf`文件，内容如下：

   ```yaml
   [root@hadoop102 ~]# vim /var/kerberos/krb5kdc/kdc.conf
   修改如下内容
   
   [kdcdefaults]
    kdc_ports = 88
    kdc_tcp_ports = 88
   
   [realms]
    EXAMPLE.COM = {
     #master_key_type = aes256-cts
     acl_file = /var/kerberos/krb5kdc/kadm5.acl
     dict_file = /usr/share/dict/words
     admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
     supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
   ```

   

2. 客户端主机（所有主机）

   修改`/etc/krb5.conf`文件：

   ```shell
   [root@hadoop102 ~]# vim /etc/krb5.conf
   [root@hadoop103 ~]# vim /etc/krb5.conf
   [root@hadoop104 ~]# vim /etc/krb5.conf
   ```

   内容如下：

   ```yaml
   # Configuration snippets may be placed in this directory as well
   includedir /etc/krb5.conf.d/
   
   [logging]
    default = FILE:/var/log/krb5libs.log
    kdc = FILE:/var/log/krb5kdc.log
    admin_server = FILE:/var/log/kadmind.log
   
   [libdefaults]
    dns_lookup_realm = false
    dns_lookup_kdc = false # 新增
    ticket_lifetime = 24h
    renew_lifetime = 7d
    forwardable = true
    rdns = false
    pkinit_anchors = FILE:/etc/pki/tls/certs/ca-bundle.crt
    default_realm = EXAMPLE.COM # 新增，kinit认证时如果没有写@后的内容，则默认为此
    #default_ccache_name = KEYRING:persistent:%{uid}
   
   # 新增
   [realms]
    EXAMPLE.COM = { # 域realm
     kdc = hadoop102 # kdc所在的主机名
     admin_server = hadoop102 # admin_server所在的主机名
    }
    
    
   [domain_realm]
   # .example.com = EXAMPLE.COM
   # example.com = EXAMPLE.COM
   ```

   

### 1.2.3 初始化KDC数据库

在服务端主机（hadoop102）执行以下命令，并根据提示输入密码。

```shell
[root@hadoop102 ~]# kdb5_util create -s
```

### 1.2.4 修改管理员权限配置文件

管理员用于管理其他的用户。

在服务端主机（hadoop102）修改`/var/kerberos/krb5kdc/kadmin5.acl`文件，内容如下：

```shell
*/admin@EXAMPLE.COM	*
```

==含义解释 TODO==

### 1.2.5 启动Kerberos相关服务

在主节点（hadoop102）启动KDC，并配置开机自启

```shell
[root@hadoop102 ~]# systemctl start krb5kdc
# 配置krb5kdc开机自启
[root@hadoop102 ~]# systemctl enable krb5kdc
```

在主节点（hadoop102）启动Kadmin，该服务为KDC数据库访问入口

```shell
[root@hadoop102 ~]# systemctl start kadmin
[root@hadoop102 ~]# systemctl enable kadmin
```

==kadmin就是之前在客户端配置文件中配置的admin_server==

### 1.2.6 创建Kerberos管理员用户

在KDC所在主机（hadoop102），执行以下命令，并按照提示输入密码

```shell
[root@hadoop102 ~]# kadmin.local -q "addpric admin/admin"
```



## 1.3 Kerberos使用概述

### 1.3.1 Kerberos数据库操作

1. 登录数据库

   - 本地登录（<span style="color:red;">无需认证</span>）

     ````shell
     [root@hadoop102 ~]# kadmin.local
     Authenticating as pricipal root/admin@EXAMPLE.COM with password.
     kadmin.local:
     ````

     

   - 远程登录（<span style="color:red;">需要进行主体认证，认证操作见下文</span>）

     ```shell
     [root@hadoop103 ~]# kadmin
     Authenticating as principal admin/admin@EXAMPLE.COM with password.
     Password for admin/admin@EXAMPLE.COM: 
     kadmin:  
     ```

     

2. 创建Kerberos主体

   登录数据库，输入以下命令，并按照提示输入密码

   ```shell
   [root@hadoop102 ~]# kadmin.local
   Authenticating as pricipal root/admin@EXAMPLE.COM with password.
   kadmin.local: addprinc test
   ```

   也可通过以下shell命令直接创建主题

   ```shell
   [root@hadoop102 ~]# kadmin.local -q "addprinc test"
   ```

   

3. 修改主体密码

   ```shell
   kadmin.local: cpw test
   ```

   

4. 查看所有主体

   ```shell
   kadmin.lcoal: list_principals
   K/M@EXAMPLE.COM
   admin/admin@EXAMPLE.COM
   kadmin/admin@EXAMPLE.COM
   kadmin/changepw@EXAMPLE.COM
   kadmin/hadoop105@EXAMPLE.COM
   kiprop/hadoop105@EXAMPLE.COM
   krbtgt/EXAMPLE.COM@EXAMPLE.COM
   ```

5. 删除主体

   ```shell
   kadmin.local: delprinc test
   ```

   

### 1.3.2 Kerberos认证操作

1. 密码认证

   用户可以使用密码认证，也可以使用keytab密钥文件认证。

   - 使用kinit进行主体认证，并按照提示输入密码

     ```shell
     [root@hadoop102 ~]# kinit test
     Password for test@EXAMPLE.COM:
     ```

   - 查看凭证

     ```shell
     [root@hadoop102 ~]# klist 
     Ticket cache: FILE:/tmp/krb5cc_0
     Default principal: test@EXAMPLE.COM
     
     Valid starting       Expires              Service principal
     10/27/2019 18:23:57  10/28/2019 18:23:57  krbtgt/EXAMPLE.COM@EXAMPLE.COM
     	renew until 11/03/2019 18:23:57
     ```

     

2. 密钥文件认证

   服务需要认证的话，使用密钥文件认证更为方便。

   - 生成主体test的keytab文件到指定目录`/root/test.keytab`。

     ```shell
     kadmin.local -q "xst -norandkey -k root/test.keytab test@EXAMPLE.COM"
     ```

     ==注意：-norandkey的作用是声明不随机生成密码，若不加该参数，会导致之前的密码失效。==

   - 使用keytab进行认证

     ```shell
     [root@hadoop102 ~]# kinit -kt /root/test.keytab
     ```

   - 查看认证凭证

     ```shell
     [root@hadoop102 ~]# klist
     Ticket cache: FILE:/tmp/krb5cc_0
     Default principal: test@EXAMPLE.COM
     
     Valid starting     Expires            Service principal
     08/27/19 15:41:28  08/28/19 15:41:28  krbtgt/EXAMPLE.COM@EXAMPLE.COM
             renew until 08/27/19 15:41:28
     ```

     

3. 销毁凭证

   ```shell
   [root@hadoop102 ~]# kdestroy
   [root@hadoop102 ~]# klist
   klist: No credentials cache found (ticket cache FILE:/tmp/krb5cc_0)
   ```

   

# 2. 创建Hadoop系统用户

为Hadoop开启Kerberos，需为不同服务准备不同的用户，启动服务时需要使用相应的用户。须在<span style="color:red;">所有节点</span>创建以下用户和用户组。

| **User:Group**    | **Daemons**                                         |
| ----------------- | --------------------------------------------------- |
| **hdfs:hadoop**   | NameNode, Secondary NameNode, JournalNode, DataNode |
| **yarn:hadoop**   | ResourceManager, NodeManager                        |
| **mapred:hadoop** | MapReduce JobHistory Server                         |

创建hadoop组

```shell
[root@hadoop102 ~]# groupadd hadoop
[root@hadoop103 ~]# groupadd hadoop
[root@hadoop104 ~]# groupadd hadoop
```

创建各用户并设置密码

```shell
[root@hadoop102 ~]# useradd hdfs -g hadoop
[root@hadoop102 ~]# echo hdfs | passwd --stdin  hdfs

[root@hadoop102 ~]# useradd yarn -g hadoop
[root@hadoop102 ~]# echo yarn | passwd --stdin yarn

[root@hadoop102 ~]# useradd mapred -g hadoop
[root@hadoop102 ~]# echo mapred | passwd --stdin mapred

[root@hadoop103 ~]# useradd hdfs -g hadoop
[root@hadoop103 ~]# echo hdfs | passwd --stdin  hdfs

[root@hadoop103 ~]# useradd yarn -g hadoop
[root@hadoop103 ~]# echo yarn | passwd --stdin yarn

[root@hadoop103 ~]# useradd mapred -g hadoop
[root@hadoop103 ~]# echo mapred | passwd --stdin mapred


[root@hadoop104 ~]# useradd hdfs -g hadoop
[root@hadoop104 ~]# echo hdfs | passwd --stdin  hdfs

[root@hadoop104 ~]# useradd yarn -g hadoop
[root@hadoop104 ~]# echo yarn | passwd --stdin yarn

[root@hadoop104 ~]# useradd mapred -g hadoop
[root@hadoop104 ~]# echo mapred | passwd --stdin mapred
```

# 3. Hadoop Kerberos配置

## 3.1 为Hadoop各服务创建Kerberos主体（Principal）

主体格式如下：ServiceName/HostName@REALM，例如 dn/hadoop102@EXAMPLE.COM

1. 各服务所需主体如下

   环境：3台节点，主机名分别为hadoop102，hadoop103，hadoop104。由于VMware虚拟机性能不够，就没有开启HDFS HA和Yarn HA。

   | **服务**           | **所在主机** | **主体（Principal）** |
   | ------------------ | ------------ | --------------------- |
   | NameNode           | hadoop102    | nn/hadoop102          |
   | DataNode           | hadoop102    | dn/hadoop102          |
   | DataNode           | hadoop103    | dn/hadoop103          |
   | DataNode           | hadoop104    | dn/hadoop104          |
   | Secondary NameNode | hadoop104    | sn/hadoop104          |
   | ResourceManager    | hadoop103    | rm/hadoop103          |
   | NodeManager        | hadoop102    | nm/hadoop102          |
   | NodeManager        | hadoop103    | nm/hadoop103          |
   | NodeManager        | hadoop104    | nm/hadoop104          |
   | JobHistory Server  | hadoop102    | jhs/hadoop102         |
   | Web UI             | hadoop102    | HTTP/hadoop102        |
   | Web UI             | hadoop103    | HTTP/hadoop103        |
   | Web UI             | hadoop104    | HTTP/hadoop104        |

2. 创建主体说明

   - 路径准备

     为服务创建的主体，需要通过密钥文件keytab文件进行认证，故需为各服务准备一个安全的路径用来存储keytab文件。

     ```shell
     [root@hadoop102 ~]# mkdir /etc/security/keytab/
     [root@hadoop102 ~]# chown -R root:hadoop /etc/security/keytab/
     [root@hadoop102 ~]# chmod 770 /etc/security/keytab/
     ```

   - 管理员主体认证

     为执行创建主体的语句，需登录Kerberos 数据库客户端，登录之前需先使用Kerberos的管理员用户进行认证，执行以下命令并根据提示输入密码。

     ```shell
     [root@hadoop102 ~]# kinit admin/admin
     ```

   - 登录数据库客户端

     ```shell
     [root@hadoop102 ~]# kadmin
     ```

   - 执行创建主体的语句

     ```shell
     kadmin:  addprinc -randkey test/test
     kadmin:  xst -k /etc/security/keytab/test.keytab test/test
     ```

     说明：

     （1）`addprinc test/test`：作用是新建主体

     `addprinc`：增加主体

     `-randkey`：密码随机，因hadoop各服务均通过keytab文件认证，故密码可随机生成

     `test/test`：新增的主体

     （2）`xst -k /etc/security/keytab/test.keytab test/test`：作用是将主体的密钥写入keytab文件

     `xst`：将主体的密钥写入keytab文件

     `-k /etc/security/keytab/test.keytab`：指明keytab文件路径和文件名

     `test/test`：主体

     （3）为方便创建主体，可使用如下命令

     ```shell
     [root@hadoop102 ~]# kadmin -p admin/admin -wadmin -q"addprinc -randkey test/test"
     [root@hadoop102 ~]# kadmin -p admin/admin -wadmin -q"xst -k /etc/security/keytab/test.keytab test/test"
     ```

     说明：

     -p：主体

     -w：密码

     -q：执行语句

     （4）操作主体的其他命令，可参考官方文档，地址如下：

     [http://web.mit.edu/kerberos/krb5-current/doc/admin/admin_commands/kadmin_local.html#commands](#commands) 

3. 创建主体

   TODO

4. x

## 3.2 修改Hadoop配置文件

## 3.3 配置HDFS使用HTTPS安全传输协议

## 3.4 配置Yarn使用LinuxContainerExecutor

# 4. 安全模式下启动Hadoop集群

## 4.1

## 4.2

## 4.3

## 4.4

## 4.5