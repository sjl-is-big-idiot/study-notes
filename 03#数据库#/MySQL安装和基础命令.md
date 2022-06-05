# 1. 介绍

# 2. 安装

安装mysql客户端
yum install mysql  -y

# 3. 配置

# 4. 启停

# 5. 常用命令



连接目标主机mysql
mysql -h192.168.43.119 -uroot -p1234

查看数据库
show databases;

使用test数据库
use test

查询dept表
select * fom dept ;

退出连接
exit;

 

## 查看mysql版本

mysql --help

mysql -h1.1.1.1 -uroot -p

mysql> status;

mysql>select version();



## 数据库

### 1.查看所有数据库容量大小

select
table_schema as '数据库',
sum(table_rows) as '记录数',
sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
from information_schema.tables
group by table_schema
order by sum(data_length) desc, sum(index_length) desc;

### 2.查看所有数据库各表容量大小

select
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024, 2) as '数据容量(MB)',
truncate(index_length/1024/1024, 2) as '索引容量(MB)'
from information_schema.tables
order by data_length desc, index_length desc;

### 3.查看指定数据库容量大小

例：查看mysql库容量大小

select
table_schema as '数据库',
sum(table_rows) as '记录数',
sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
from information_schema.tables
where table_schema='mysql';

### 4.查看指定数据库各表容量大小

例：查看mysql库各表容量大小

select
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024, 2) as '数据容量(MB)',
truncate(index_length/1024/1024, 2) as '索引容量(MB)'
from information_schema.tables
where table_schema='mysql'
order by data_length desc, index_length desc;

## 表

TODO



## 用户

### 查看mysql用户信息

```sql
SELECT User, Host, Password FROM mysql.user;
```

查看某mysql用户的所有信息

```sql
SELECT * FROM mysql.user WHERE user='root'\G
```

### mysql修改用户密码

```sql
--查看当前的mysql用户信息
select Host,User,Password from mysql.user;

UPDATE mysql.user SET password=PASSWORD("yourpassword") WHERE user='root';
flush privileges;

--查看当前的mysql用户信息
select Host,User,Password from mysql.user;
```



### mysql创建新用户

```sql
create user kzt_zt identified by 'yourpassword';
GRANT ALL PRIVILEGES ON *.* TO 'kzt_zt'@'%' IDENTIFIED BY 'yourpassword' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'kzt_zt'@'localhost' IDENTIFIED BY 'yourpassword' WITH GRANT OPTION;
flush privileges;
```



## 索引和主键

查看某表的索引

```sql
SHOW INDEX FROM 表名;
```

查看某表的键

```sql
show keys from 表名;
--或者
show full columns from 表名;
```



查看某表的主键

```sql
SELECT t.TABLE_NAME,
       t.CONSTRAINT_TYPE,
       c.COLUMN_NAME,
       c.ORDINAL_POSITION
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS AS t,
     INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS c
WHERE t.TABLE_NAME = c.TABLE_NAME
  AND t.CONSTRAINT_TYPE = 'PRIMARY KEY'
  AND t.TABLE_NAME='[$Table_Name]'
  AND t.TABLE_SCHEMA='[$DB_Name]';
```

## 存储过程

查看存储过程状态

```shell
SHOW PROCEDURE STATUS LIKE '存储过程名' \G
```

查看存储过程创建语句

```sql
SHOW CREATE PROCEDURE <存储过程名>;
```

## 权限

参考自：https://blog.csdn.net/weixin_42109012/article/details/102684153

### MySQL权限级别介绍

1. 全局——可以管理整个MySQL
2. 库——可以管理指定的数据库
3. 表——可以管理指定数据库的指定表
4. 字段——可以管理指定数据库的指定表的指定字段

权限存储在mysql库的`user, db, tables_priv, columns_priv, procs_priv`这几个系统表中，待MySQL实例启动后就加载到[内存](https://so.csdn.net/so/search?q=内存&spm=1001.2101.3001.7020)中。

**MySQL查看用户权限**

```sql
SELECT * FROM mysql.user WHERE user='root'\G
```

\g 相当于’;’
\G使每个字段打印到单独的行，也有’;'的作用

**用户信息：授权对象，连接用户名，用户密码**

```shell
Host: %		# 授权用户，% 代表所有
User: root	# 用户名
authentication_string: *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9	#密码，MD5加密
```

**授权信息**

```shell
Select_priv：确定用户是否可以通过SELECT命令选择数据 
Insert_priv：确定用户是否可以通过INSERT命令插入数据 
Update_priv：确定用户是否可以通过UPDATE命令修改现有数据 
Delete_priv：确定用户是否可以通过DELETE命令删除现有数据 
Create_priv：确定用户是否可以创建新的数据库和表 
Drop_priv：确定用户是否可以删除现有数据库和表 
Reload_priv：确定用户是否可以执行刷新和重新加载MySQL所用各种内部缓存的特定命令，包括日志、权限、主机、查询和表 
Shutdown_priv：确定用户是否可以关闭MySQL服务器，将此权限提供给root账户之外的任何用户时，都应当非常谨慎 
Process_priv：确定用户是否可以通过SHOW 
File_priv：确定用户是否可以执行SELECT INTO OUTFILE和LOAD DATA INFILE命令 
Grant_priv：确定用户是否可以将已经授予给该用户自己的权限再授予其他用户，例如，如果用户可以插入、选择和删除foo数据库中的信息，并且授予了GRANT权限，则该用户就可以将其任何或全部权限授予系统中的任何其他用户 
References_priv：目前只是某些未来功能的占位符，现在没有作用 
Index_priv：确定用户是否可以创建和删除表索引 
Alter_priv：确定用户是否可以重命名和修改表结构 
Show_db_priv：确定用户是否可以查看服务器上所有数据库的名字，包括用户拥有足够访问权限的数据库，可以考虑对所有用户禁用这个权限，除非有特别不可抗拒的原因 
Super_priv：确定用户是否可以执行某些强大的管理功能，例如通过KILL命令删除用户进程，使用SET GLOBAL修改全局MySQL变量，执行关于复制和日志的各种命令 
Create_tmp_table_priv：确定用户是否可以创建临时表 
Lock_tables_priv：确定用户是否可以使用LOCK 
Execute_priv：确定用户是否可以执行存储过程，此权限只在MySQL 5.0及更高版本中有意义 
Repl_slave_priv：确定用户是否可以读取用于维护复制数据库环境的二进制日志文件，此用户位于主系统中，有利于主机和客户机之间的通信 
Repl_client_priv：确定用户是否可以确定复制从服务器和主服务器的位置 
Create_view_priv：确定用户是否可以创建视图，此权限只在MySQL 5.0及更高版本中有意义 
Show_view_priv：确定用户是否可以查看视图或了解视图如何执行，此权限只在MySQL 5.0及更高版本中有意义 Create_routine_priv：确定用户是否可以更改或放弃存储过程和函数，此权限是在MySQL 5.0中引入的 Alter_routine_priv：确定用户是否可以修改或删除存储函数及函数，此权限是在MySQL 5.0中引入的 Create_user_priv：确定用户是否可以执行CREATE 
Event_priv：确定用户能否创建、修改和删除事件，这个权限是MySQL 5.1.6新增的 
Trigger_priv：确定用户能否创建和删除触发器，这个权限是MySQL 5.1.6新增的
Create_tablespace_priv: 创建表的空间
```

**权限表**

![在这里插入图片描述](MySQL安装和基础命令.assets/20191022152019290.png)

### 授权

**每次更新权限后记得刷新权限**

```sql
FLUSH PRIVILEGES;
```

格式：

```python
GRANT
  [权限] 
ON [库.表] 
TO [用户名]@[IP] 
IDENTIFIED BY [密码] 
# WITH GRANT OPTION;
```

GRANT命令说明：
(1)`ALL PRIVILEGES` 表示所有权限，你也可以使用select、update等权限。
(2)`ON` 用来指定权限针对哪些库和表。
(3)`*.*` 中前面的*号用来指定数据库名，后面的*号用来指定表名。
(4)`TO` 表示将权限赋予某个用户。
(5)`@` 前面表示用户，@后面接限制的主机，可以是IP、IP段、域名以及%，%表示任何地方。
(6)`IDENTIFIED BY` 指定用户的登录密码。
(7)`WITH GRANT OPTION` 这个选项表示该用户可以将自己拥有的权限授权给别人。

<font style="color:red">***注意：经常有人在创建操作用户的时候不指定WITH GRANT OPTION选项导致后来该用户不能使用GRANT命令创建用户或者给其它用户授权。***</font>

<font style="color:red">***备注：可以使用GRANT重复给用户添加权限，权限叠加，比如你先给用户添加一个select权限，然后又给用户添加一个insert权限，那么该用户就同时拥有了select和insert权限。***</font>

**1、全局授权（直接把 root 限制主机改为 %，任意主机）**
因为 root 是数据库用户默认最高权限

```python
UPDATE mysql.user SET user.Host='%' where user.User='root';
FLUSH PRIVILEGES;
```

**当然也可以新建一个用户，给与全部权限**

```python
GRANT
  ALL PRIVILEGES
ON *.*
TO admin@'175.155.59.133'
IDENTIFIED BY 'admin';
```

**这样你就可以在远程连接到该数据库，且获取全部权限。**
![在这里插入图片描述](MySQL安装和基础命令.assets/20191022163322119.png)
**2、单个数据库授权**

只给175.155.59.133这个 IP 赋给 ctrip 数据库 查询的权限，用户：ctrip，密码：ctrip

```python
GRANT
  select
ON ctrip.*
TO ctrip@'175.155.59.133'
IDENTIFIED BY 'ctrip';
```

声明：
（1）用ipconfig查询出来的IP，那是局域网的，这么设置只能局域网内使用，
（2）要想服务器和本机连接，IP 必须是网关的IP，推荐使用 https://www.ipip.net/ 查询自己的 IP。
![在这里插入图片描述](MySQL安装和基础命令.assets/20191022164036376.png)
**可以看到本机连接有 ctrip 数据库和一个数据库本身库，而看不到其他库。注意此时可以看到两个表。**
![在这里插入图片描述](MySQL安装和基础命令.assets/20191022165804360.png)
![在这里插入图片描述](MySQL安装和基础命令.assets/2019102216561327.png)

**3、单个数据库单个表授权**

```python
GRANT
  select
ON ctrip.t_plane
TO ctrip@'175.155.59.133'
IDENTIFIED BY 'ctrip';
```

**可以看到只有一个表显示出来，注意现在字段**
![在这里插入图片描述](MySQL安装和基础命令.assets/2019102217025989.png)
**4、单个数据库单个表授权某些字段授权**

```python
GRANT
  select(id,EN)
ON ctrip.t_plane
TO ctrip@'175.155.59.133'
IDENTIFIED BY 'ctrip';
```

**这样做的话，我们是连表都是打不开的，只能通过查询语句，查出对我们开放的字段**
![在这里插入图片描述](MySQL安装和基础命令.assets/20191022172923326.png)

### 四、收回权限、删除用户

**1、收回权限**
格式：

```python
REVOKE
  [权限] 
ON [库.表] 
FROM [用户名]@[IP];
```

操作：

```python
REVOKE
  select(id,EN)
ON ctrip.t_plane
FROM ctrip@'175.155.59.133';
```

**2、删除用户**
格式：

```python
DROP USER [用户名]@[IP];
```

操作：

```python
DROP USER ctrip@'175.155.59.133';
```

## 引擎

### 联邦引擎

登录mysql查看是否开启了联邦引擎

```sql
show engines;
```

![img](MySQL安装和基础命令.assets/993337-20180704084123390-1139277077.png)

在mysql安装目录下的`my.cnf`配置文件中的`[mysqld]`部分下添加`federated`。

**如何利用联邦引擎建立与远程mysql库表的映射**

在服务器A上有MySQL数据库test_a,在服务器B上有MySQL数据库test_b。现在需要将test_a库中的user表数据映射到数据库test_b中。此时需要在数据库test_b中建立表user，注意ENGINE和CONNECTION。

```sql
CREATE TABLE user (
  id int(11) NOT NULL,
  name varchar(30) NOT NULL,
  age int(11) NOT NULL,
  PRIMARY KEY (id)
) ENGINE=FEDERATED 
CONNECTION='mysql://test:123456@192.168.1.5:3306/test_a/user'
DEFAULT CHARSET=utf8;
```

上面链接中test是链接数据库用户名称；123456是密码；192.168.1.5是数据库服务器ip；3306是数据库服务器端口；test_a是数据库名称；user是数据库表名称。

连接字符串的实例：

```sql
CONNECTION='mysql://username:password@hostname:port/database/tablename'
CONNECTION='mysql://username@hostname/database/tablename'
CONNECTION='mysql://username:password@hostname/database/tablename'
```



服务器A上MySQL数据库test_a设置可以远程访问，并给test用户分配相关表的读写权限。

此时，修改test_b中的user表后，就可以在test_a中的user表中看到相关改动；同理，修改test_a中的user表后，就可以在test_b中的user表中看到相关改动。

[MySQL开启federated引擎实现数据库表映射](https://www.cnblogs.com/shuilangyizu/p/9261567.html)



**查看mysql中有哪些联邦表**

```sql
SELECT TABLE_SCHEMA, TABLE_NAME, engine FROM INFORMATION_SCHEMA.TABLES WHERE engine='FEDERATED';
```





## 主从同步

### mysql查看主从同步状态的方法

查看主库运行状态

```sql
show master status\G
```


查看从库运行状态

```sql
show slave status\G
```

> -- 负责把主库bin日志(Master_Log)内容投递到从库的中继日志上(Relay_Log)
> Slave_IO_Running: Yes
> -- 负责把中继日志上的语句在从库上执行一遍
> Slave_SQL_Running: Yes
> -- Yes：表示正常， No：表示异常

停止主从同步，登录到从库的mysql

```sql
stop slave;
```

然后查看从库运行状态中的`Slave_IO_Running`和`Slave_SQL_Running`应该都为`NO`才对。



## 编码和字符集

[mysql怎么查看表的字符集](https://m.php.cn/article/460632.html)

[怎么查询mysql的字符集](https://m.php.cn/article/488196.html)

**查看MYSQL数据库服务器和数据库字符集**

```sql
方法一：show variables like '%character%';
方法二：show variables like 'collation%';
```

修改数据库的编码格式

方法一：

```sql
--命令为：
set character% = utf8；

--例如：
set character_set_client =utf8;
```

**查看MYSQL所支持的字符集**

```sql
show charset;
```

**查看库的字符集**

```sql
show database status from 库名 like 表名;
```

```sql
show create database shiyan;
```

**查看表的字符集**

```sql
show table status from 库名 like 表名;
```

**查看表中所有字段的字符集**

```sql
show full columns from 表名;
```



## 参数配置

#### general_log

[mysql general log使用介绍](https://blog.csdn.net/lanyang123456/article/details/110727425)

general log 是MySQL 日志的一种，它会记录MySQL执行的每条SQL，非常详细。

但对MySQL性能有影响，为了性能考虑，一般general log不会开启，除非排查问题。

开启general log有两种方式。

**查看当前`general_log`的值**

```sql
show variables like 'general%';
```

**临时修改`general_log`**

```sql
-- 开启后，执行的所有sql，都会记录到general_log_file文件中。
set global general_log=on;
-- 关闭
set global general_log=off;
```

这种方式对`general_log`的修改，重启mysql服务后，会失效。

**永久修改`general_log`**

`/etc/my.cnf`配置文件中，增加配置：

```sql
[mysqld]
general_log = 1
general_log_file = /tmp/general.log
```

重启mysql后生效。

*注：开启general log一般就是为了排查问题，如果不再使用，记得及时关闭，以免影响性能。*

### sql_mode

**查看session级别的`sql_mode`**

```sql
select @@session.sql_mode;
```

**查看全局级别的`sql_mode`**

```sql
select @@global.sql_mode;
```

**修改session级别的`sql_mode`**

```sql
set @@session.sql_mode='xx_mode'
--或
set session sql_mode='xx_mode'
```

session均可省略，默认session，仅对当前会话有效

**修改全局级别的`sql_mode`**

```sql
set global sql_mode='xx_mode';
--或
set @@global.sql_mode='xx_mode';
```

*注：全局修改的话，需高级权限，仅对下次连接生效，不影响当前会话，且MySQL重启后失效，因为MySQL重启时会重新读取配置文件里对应值，如果需永久生效需要修改配置文件里的值。*

```shell
vi /etc/my.cnf

[mysqld]

sql-mode = "xx_mode"
```

保存退出，重启服务器，即可永久生效



# mysql问题

### mysql忘记root用户密码，无法登录了

这种情形是十分有可能遇到的，比如做mysql物理备份数据恢复后，目标端的mysql.user表中的数据也是源端mysql的数据，即造成root用户的密码改动了，可能会出现无法登录的情况

**解决方案：**

> 让mysql跳过密码认证即可，有两种实现方式：
>
> 方式一：[mysql实现不用密码登录的实例方法](https://www.jb51.net/article/194448.htm)
>
> a、停止mysql服务
>
> ```shell
> /etc/init.d/mysqld stop
> #或
> systemctl stop mysqld
> ```
>
> b、跳过密码验证
>
> ```shell
> /usr/bin/mysqld_safe --skip-grant-tables
> ```
>
> 或
>
> ```shell
> mysqld_safe --skip-grant-tables
> ```
>
> 跳过权限表启动mysql。
>
> c、登录mysql服务端，修改root用户密码，并刷新权限
>
> ```sql
> --登录mysql
> mysql
> 
> --修改root用户密码
> UPDATE mysql.user SET password=PASSWORD("yourpassword") WHERE user='root';
> flush privileges;
> ```
>
> d、重启mysql
>
> 方式二：[MySQL设置免密登录](https://blog.51cto.com/u_12902538/3726893)
>
> a、修改/etc/my.cnf中[mysqld]部分内容
>
> ```shell
> vim /etc/my.cnf
> 
> 在[mysqld]最后添加：skip-grant-tables
> ```
>
> b、重启MySQL
> c、直接mysql进入mysql服务端，修改root用户密码，并刷新权限
>
> ```sql
> --登录mysql
> mysql
> 
> --修改root用户密码
> UPDATE mysql.user SET password=PASSWORD("yourpassword") WHERE user='root';
> flush privileges;
> ```
>
> d、退出，删掉/etc/my.cnf的skip-grant-tables
>
> ```shell
> vim /etc/my.cnf
> 
> 删掉skip-grant-tables
> ```
>
> e、重启mysql

## mysql报错：ORDER BY clause is not in GROUP BY clause and contains nonaggregated column

https://blog.csdn.net/zx1293406/article/details/103401803

**问题原因：**

`sql_mode`设置为`only_full_group_by`，导致出现此报错

**解决方案：**

> 修改sql_mode=空。不用重启mysql