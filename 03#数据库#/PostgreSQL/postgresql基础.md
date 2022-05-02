# 1. 介绍

// TODO

#					2. 安装						#
#################################################

## 2.1 离线安装pg
https://blog.csdn.net/u010177412/article/details/82150207

## 2.2 yum安装pg

移除之前低版本的pg

```shell
yum remove postgresql*
```

装上 安装pg所需的依赖

```shel
yum install zlib-devel
yum install readline-devel
```

yum安装

```shell
yum install postgresql
```



## 2.3 编译，安装pg

```shell
cd /opt/postgresql
./configure --prefix=/usr/local/postgresql
make && make install
```

启动

```shell
/usr/pgsql-13/bin/pg_ctl -D /home/pgsql-13/data -l logfile start  #启动
/usr/pgsql-13/bin/pg_ctl -D /home/pgsql-13/data -l logfile stop  #停止
/usr/pgsql-13/bin/pg_ctl -D /home/pgsql-13/data -l logfile reload  #重启
```









# 3. pg常用命令

```sql
1、列举数据库：\l或SELECT datname FROM pg_database;
2、选择或切换数据库：\c 数据库名
3、查看该某个库中的所有表：\dt或\d 数据库名
4、查看某个库中的某个表结构：\d 表名
5、查看某个库中某个表的记录：select * from apps limit 1;
6、显示字符集：\encoding
7、查看帮助：help
8、退出psgl：\q
9、password test // 重新设置用户test的密码,然后需要 \q退出后才生效
10、创建用户：CREATE USER test WITH PASSWORD '*****';
11、删除用户：drop User 用户名
12、给用户设置密码：alter user test password ‘123456’；

1.创建数据库：
create database [数据库名];
2。删除数据库：
drop database [数据库名]; 
3.创建表：
create table ([字段名1] [类型1] ;,[字段名2] [类型2],......<,primary key (字段名m,字段名n,...)>;);
4.在表中插入数据：
insert into 表名 ([字段名m],[字段名n],......) values ([列m的值],[列n的值],......);
5.查看表内容:
select * from student;
6.重命名一个表：
alter table [表名A] rename to [表名B];
7.删除一个表：
drop table [表名]; 
8.在已有的表里添加字段：
alter table [表名] add column [字段名] [类型];
9.删除表中的字段：
alter table [表名] drop column [字段名];
10.重命名一个字段： 
alter table [表名] rename column [字段名A] to [字段名B];
11.给一个字段设置缺省值： 
alter table [表名] alter column [字段名] set default [新的默认值];
12.去除缺省值： 
alter table [表名] alter column [字段名] drop default;
13.修改表中的某行某列的数据：
update [表名] set [目标字段名]=[目标值] where [该行特征];
14.删除表中某行数据：
delete from [表名] where [该行特征];
delete from [表名];    // 删空整个表
```



## 函数

#查看有哪些函数

```sql
SELECT
  pg_proc.proname AS "函数名称",
  pg_type.typname AS "返回值数据类型",
  pg_proc.pronargs AS "参数个数"
FROM
  pg_proc
    JOIN pg_type
   ON (pg_proc.prorettype = pg_type.oid)
WHERE
  pg_type.typname != 'void'
  AND pronamespace = (SELECT pg_namespace.oid FROM pg_namespace WHERE nspname = 'public');
```

了解一共有哪些自己可见的函数

```
\df
```



## 用户

创建用户

```sql
\h create user 查看帮助
create user 用户名 with superuser password '密码'; 

在shell中用命令createuser --help查看帮助 
createuser 用户名  -s  -P '密码'
```

列出PostgreSQL用户

```sql
\du
```

### PostgreSQL的用户、角色和权限管理

[PostgreSQL的用户、角色和权限管理](https://blog.csdn.net/eagle89/article/details/80363365)

> Pg权限分为两部分，一部分是“系统权限”或者数据库用户的属性，可以授予role或user（**两者区别在于login权限**）；一部分为数据库对象上的操作权限。
>
> 超级用户不做权限检查，其它走**acl**。
>
> 对于数据库对象，开始只有所有者和超级用户可以做任何操作，其它走acl。在pg里，对acl模型做了简化，组和角色都是role，**用户和角色的区别是角色没有login权限**。

CREATE ROLE创建的用户默认不带LOGIN属性，而CREATE USER创建的用户默认带有LOGIN属性

```sql
 postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

postgres=# CREATE ROLE test_user_1;    /*默认不带LOGIN属性*/
CREATE ROLE
postgres=# CREATE user test_user_2;  /*默认具有LOGIN属性*/
CREATE ROLE
postgres=# \du
                                    List of roles
  Role name  |                         Attributes                         | Member of
-------------+------------------------------------------------------------+-----------
 postgres    | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 test_user_1 | Cannot login                                               | {}
 test_user_2 |                                                            | {}
```

在创建用户时赋予角色属性

```sql
postgres=# CREATE  ROLE test_user_3 CREATEDB;  /*具有创建数据库的属性*/ 
CREATE ROLE
postgres=# \du
                                    List of roles
  Role name  |                         Attributes                         | Member of
-------------+------------------------------------------------------------+-----------
 postgres    | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 test_user_1 | Cannot login                                               | {}
 test_user_2 |                                                            | {}
 test_user_3 | Create DB, Cannot login                                    | {}
```

给已存在用户赋予各种权限

```sql
postgres=# ALTER ROLE test_user_3 WITH LOGIN; /*赋予登录权限*/ 
ALTER ROLE
postgres=#  ALTER ROLE test_user_4 WITH CREATEROLE;/*赋予创建角色的权限*/ 

ALTER ROLE
postgres=# \du
                                    List of roles
  Role name  |                         Attributes                         | Member of
-------------+------------------------------------------------------------+-----------
 postgres    | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 test_user_1 | Cannot login                                               | {}
 test_user_2 |                                                            | {}
 test_user_3 | Create DB                                                  | {}
 test_user_4 | Create role, Create DB, Cannot login                       | {}
postgres=#  SELECT * FROM pg_roles;  
       rolname        | rolsuper | rolinherit | rolcreaterole | rolcreatedb | rolcanlogin | rolreplication | rolconnlimit | rolpassword | rolvaliduntil | rolbypassrls | rolconfig |  oid  
----------------------+----------+------------+---------------+-------------+-------------+----------------+--------------+-------------+---------------+--------------+-----------+-------
 pg_signal_backend    | f        | t          | f             | f           | f           | f              |           -1 | ********    |               | f            |           |  4200
 test_user_3          | f        | t          | f             | t           | t           | f              |           -1 | ********    |               | f            |           | 16389
 postgres             | t        | t          | t             | t           | t           | t              |           -1 | ********    |               | t            |           |    10
 pg_read_all_stats    | f        | t          | f             | f           | f           | f              |           -1 | ********    |               | f            |           |  3375
 pg_monitor           | f        | t          | f             | f           | f           | f              |           -1 | ********    |               | f            |           |  3373
 test_user_2          | f        | t          | f             | f           | t           | f              |           -1 | ********    |               | f            |           | 16388
 test_user_1          | f        | t          | f             | f           | f           | f              |           -1 | ********    |               | f            |           | 16387
 test_user_4          | f        | t          | t             | t           | f           | f              |           -1 | ********    |               | f            |           | 16390
 pg_read_all_settings | f        | t          | f             | f           | f           | f              |           -1 | ********    |               | f            |           |  3374
 pg_stat_scan_tables  | f        | t          | f             | f           | f           | f              |           -1 | ********    |               | f            |           |  3377
(10 rows)
postgres=# ALTER ROLE pg_test_user_4 WITH PASSWORD '654321';/*修改密码*/ 
ALTER ROLE 
postgres=# ALTER ROLE pg_test_user_4 VALID UNTIL 'JUL 7 14:00:00 2012 +8'; /*设置角色的有效期* 
ALTER ROLE  
```



## 触发器

了解一张表有哪些触发器

```
\d table_name
```

了解触发器所引用函数的定义

```
\sf function_name
```



## 连接

```shell
/usr/local/postgresql/bin/psql -U root -h 10.225.67.145 -p 5432 -d template1
```

-u：表示登入的用户

-h：表示pg服务器ip

-p：表示pg服务器监听的端口

-d：表示进入的数据库名

## 数据库

列出可用的数据库

```sql
\l;
```

切换到某个数据库

```sql
\c 数据库名;
```

查看pg数据库的数据量大小

```sql


template1=# select pg_database_size('emap_npls');
 pg_database_size 
------------------
         49886328
(1 row)

-- 或者

template1=# select pg_database.datname, pg_size_pretty (pg_database_size(pg_database.datname)) AS size from pg_database;
  datname  |  size   
-----------+---------
 template1 | 6493 kB
 template0 | 6377 kB
 postgres  | 6493 kB
 emap      | 784 MB
 emap_ksf  | 11 MB
 emap_npls | 48 MB
(6 rows)
```

修改某schema所属用户

```sql
alter schema emap owner to root;
```





## 表

查看某数据库下有哪些表

```sql
\d;
-- 或者
select tablename from pg_tables where schemaname='public'；
```

查看表结构

```sql
\d 表名;
-- 或者
select * from pg_tables 
```

列出某个数据库中的表

```sql
\dt
```

列出表权限

```sql
\z
```

列出所有可用的元命令

```sql
\?
```

列出所有可用的SQL命令

```sql
\h
```

退出数据库

```sql
\q
```

查看所有表的大小

```sql
select relname, pg_size_pretty(pg_relation_size(relid)) from pg_stat_user_tables where schemaname='public' order by pg_relation_size(relid) desc;

                    relname                     | pg_size_pretty 
------------------------------------------------+----------------
 dim_sight_area                                 | 184 kB
 dim_page_type                                  | 8192 bytes
 wirelessapi_log_2014_08_08                     | 0 bytes
 trace_log_2014_08_04                           | 0 bytes
 dm_mobile                                      | 0 bytes
 trace_log_2014_07_14                           | 0 bytes
```



删除表中数据，但保留表结构。

```sql
delete from tableName;

truncate table tableName;
```





## 索引



按顺序查看索引

```sql
select indexrelname, pg_size_pretty(pg_relation_size(relid)) from pg_stat_user_indexes where schemaname='public' order by pg_relation_size(relid) desc;

                        indexrelname                        | pg_size_pretty 
------------------------------------------------------------+----------------
 pk_dim_sight_area                                          | 184 kB
 idx_area_dim_sight_area                                    | 184 kB
 idx_city_dim_sight_area                                    | 184 kB
 idx_country_dim_sight_area                                 | 184 kB
 idx_region_dim_sight_area                                  | 184 kB
 pk_dim_page_type                                           | 8192 bytes
 cpc_supplier_sight_daily_pkey                              | 0 bytes
```

查看当前数据库下有哪些索引：

```sql
select * from pg_indexes where tablename = 'pg_index';
```



## 逻辑备份

[postgreSQL数据库备份和恢复（pg_dump和pg_restore）](https://blog.csdn.net/moshowgame/article/details/117899296)

> 逻辑备份一般用`pg_dump`或者`pg_dumpall`
> –`pg_dumpall`将数据库集群全部逻辑转储到一个文件中。
> –`pg_dump`命令可以选择一个数据库或部分表进行备份。



<font style="color:red;">*注：`pg_dump`生成的备份文件可以是一个`SQL脚本`文件或`归档dump`文件。归档文件格式必须和`pg_restore`一起使用，因为是属于postgress的特殊封装。*</font>



```sql
--备份SQL格式
pg_dump -h 192.168.2.1 -U postgress QSR > /backup/QSR_bak202106.sql

--备份DUMP格式，需要-Fc
pg_dump -Fc -h 192.168.2.1 -U postgress QSR >/backup/QSR_bak202106.dump

--备份表，用-t指定
pg_dump -t test_1 mydb>/backup/test_202106.sql

--备份某个模式所有表
pg_dump -t 'qlik.*' QSR> /backup/QSR_bak202106.sql

--备份某个模式emp开头表，排除一张表
pg_dump -t 'qlik.t*' -T qlik.test_1 QSR > /backup/QSR_bak202106.sql

--迁移，建库（大写的C是create建库模式）
pg_dump -h 192.168.2.1 -U mydbser mydb -Fc >/backup/QSR_bak202106.dump
pg_restore -h 192.168.2.1 -U postgres -C -d postgres /backup/QSR_bak202106.dump

--不迁移，本机直接恢复（小写的c是clean干净模式）
pg_restore -h 192.168.2.1 -U postgres -c -d postgres /backup/QSR_bak202106.dump

--迁移，不建库（使用template0）
createdb -T template0 QSR2
pg_restore -d QSR2 /backup/QSR_bak202106.dump

--快照备份与检查
lvcreate -s -n snap201210614 /backup/pglvc202106 -L 240M
lvs
```



通过pg_dump导出数据和表结构等

```shell
pg_dump -U postgres emap> emap-all-20220430.sql
```

通过psql恢复pg_dump导出的数据

```shell
nohup /usr/local/postgresql/bin/psql  -U root -h 10.225.67.145 -p 5432 -d emap -f emap.sql > load.log 2>&1

-- 或者
/usr/local/postgresql/bin/psql -v ON_ERROR_STOP=1 -U root -h 10.225.67.145 -p 5432 -d emap -f emap.sql
```

`ON_ERROR_STOP=1`：表示遇到错误就停止导出

通过pg_restore恢复pg_dump导出的数据

```shell
/usr/local/postgresql/bin/pg_restore  -c -F d -U root -h 10.225.67.145 -p 5432 -d emap -f emap.sql > load.log
```

**只导出pg某数据库中的函数**

只导出表结构

```sql
/home/postgres/pgsql/bin/pg_dump   -Upostgres  test -Fc -v -s -f test.dmp 
```

从上面的逻辑备份文件中提取出function名

```sql
/home/postgres/pgsql/bin/pg_restore -l test.dmp | grep FUNCTION > funcion_list
```

获取指定函数的实现

```sql
/home/postgres/pgsql/bin/pg_restore -L ./funcion_list  test.dmp > function_list.sql
```

去掉注释

```sql
grep -v '\-\-' function_list.sql > function_list_implements.sql
```





