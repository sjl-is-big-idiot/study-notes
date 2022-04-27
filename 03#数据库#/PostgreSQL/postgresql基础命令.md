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









# 3. pg常用命令

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



## 逻辑备份

通过psql恢复pg_dump导出的数据

```shell
nohup /usr/local/postgresql/bin/psql  -U root -h 10.225.67.145 -p 5432 -d emap -f emap.dmp > load.log 2>&1

-- 或者
/usr/local/postgresql/bin/psql -v ON_ERROR_STOP=1 -U root -h 10.225.67.145 -p 5432 -d emap -f emap.dmp
```

`ON_ERROR_STOP=1`：表示遇到错误就停止导出

通过pg_restore恢复pg_dump导出的数据

```shell
/usr/local/postgresql/bin/pg_restore  -c -F d -U root -h 10.225.67.145 -p 5432 -d emap -f emap.dmp > load.log
```







