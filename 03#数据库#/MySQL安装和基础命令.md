

安装mysql客户端
yum install mysql  -y

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