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

 

# 本文介绍MySQL查看数据库表容量大小的命令语句，提供完整查询语句及实例，方便大家学习使用。

查看mysql版本

mysql --help

mysql -h1.1.1.1 -uroot -p

mysql> status;

mysql>select version();



1.查看所有数据库容量大小

select
table_schema as '数据库',
sum(table_rows) as '记录数',
sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
from information_schema.tables
group by table_schema
order by sum(data_length) desc, sum(index_length) desc;

2.查看所有数据库各表容量大小

select
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024, 2) as '数据容量(MB)',
truncate(index_length/1024/1024, 2) as '索引容量(MB)'
from information_schema.tables
order by data_length desc, index_length desc;

3.查看指定数据库容量大小

例：查看mysql库容量大小

select
table_schema as '数据库',
sum(table_rows) as '记录数',
sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
from information_schema.tables
where table_schema='mysql';

4.查看指定数据库各表容量大小

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

5. 查看编码

show variables like 'character%';
	2.修改数据库的编码格式

	方法一：命令为：set character% = utf8；
	
	例如：set character_set_client =utf8;


6. mysql查看主从同步状态的方法
	-- 查看主库运行状态
	mysql> show master status\G
	-- 查看从库运行状态
	mysql> show slave status\G
	
	-- 负责把主库bin日志(Master_Log)内容投递到从库的中继日志上(Relay_Log)
	Slave_IO_Running: Yes
	-- 负责把中继日志上的语句在从库上执行一遍
	Slave_SQL_Running: Yes
	-- Yes：表示正常， No：表示异常
	
7. mysql修改用户密码

   UPDATE mysql.user SET password=PASSWORD("Dicosp@ss123!") WHERE user='root';

   select Host,User,Password from mysql.user;

8. mysql创建新用户

create user kzt_zt identified by 'Syk^ziuOOcsLCaw9';
GRANT ALL PRIVILEGES ON *.* TO 'kzt_zt'@'%' IDENTIFIED BY 'Syk^ziuOOcsLCaw9' WITH GRANT OPTION;

GRANT ALL PRIVILEGES ON *.* TO 'kzt_zt'@'localhost' IDENTIFIED BY 'Syk^ziuOOcsLCaw9' WITH GRANT OPTION;

flush privileges;



MySQL开启联邦引擎

登录mysql查看是否开启了联邦引擎

show engines;

![img](MySQL安装和基础命令.assets/993337-20180704084123390-1139277077.png)

在mysql安装目录下的my.cnf配置文件中的[mysqld]部分下添加federated。

如何利用联邦引擎建立与远程mysql库表的映射。

在服务器A上有MySQL数据库test_a,在服务器B上有MySQL数据库test_b。现在需要将test_a库中的user表数据映射到数据库test_b中。此时需要在数据库test_b中建立表user，注意ENGINE和CONNECTION。

CREATE TABLE user (
  id int(11) NOT NULL,
  name varchar(30) NOT NULL,
  age int(11) NOT NULL,
  PRIMARY KEY (id)
) ENGINE=FEDERATED 
CONNECTION='mysql://test:123456@192.168.1.5:3306/test_a/user'
DEFAULT CHARSET=utf8;

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