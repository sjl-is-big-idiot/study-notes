```shell
# 下载mysql 5.6.37二进制包 

wget https://cdn.mysql.com/archives/mysql-5.6/mysql-5.6.37-linux-glibc2.12-x86_64.tar.gz


[root@VM-108-19-centos ~]# rpm -qa |grep mariadb
mariadb-libs-5.5.65-1.el7.x86_64
[root@VM-108-19-centos ~]# rpm -e --nodeps mariadb-libs-5.5.65-1.el7.x86_64
[root@VM-108-19-centos ~]# rpm -qa |grep mariadb
[root@VM-108-19-centos ~]# 

# 安装mysql
wget https://cdn.mysql.com/archives/mysql-5.6/mysql-5.6.37-linux-glibc2.12-x86_64.tar.gz
rpm -e --nodeps mariadb-libs-5.5.65-1.el7.x86_64
mkdir -p /data/mysqldata
useradd mysql -s /sbin/nologin -M
chown -R mysql:mysql /data/mysqldata
tar -xf mysql-5.6.37-linux-glibc2.12-x86_64.tar.gz -C /usr/local/
cd /usr/local/
mv mysql-5.6.37-linux-glibc2.12-x86_64/ mysql
chown -R mysql:mysql mysql
yum install libaio* libnuma* -y
yum install -y 'perl(Data::Dumper)'
cd mysql



# vim /etc/my.cnf 
[client] 
default-character-set=utf8mb4
socket=/usr/local/mysql/data/mysql.sock

[mysql] 
default-character-set=utf8mb4
socket=/usr/local/mysql/data/mysql.sock
#wait_timeout=86400

[mysqld]
datadir=/data/mysqldata
log-error=/data/mysqldata/mysql.log
socket=/usr/local/mysql/data/mysql.sock
symbolic-links=0
skip-name-resolve

port = 3307 
basedir=/usr/local/mysql
max_connections=2500

character-set-client-handshake=FALSE 
character-set-server=utf8mb4 
collation-server=utf8mb4_unicode_ci 
init_connect='SET NAMES utf8mb4'

default-storage-engine=INNODB
lower_case_table_name=1
max_allowed_packet=16M
interactive_timeout=600
wait_timeout=600
----------------------------------

## 添加主从配置-主节点
log-bin = mysql-bin
binlog_format = mixed
server-id =1
innodb-file-per-table =ON
skip_name_resolve=ON

## 添加主从配置-从节点
relay-log=relay-log
relay-log-index=relay-log.index
server-id=12
innodb_file_per_table=ON
skip_name_resolve=ON


#初始化数据库

scripts/mysql_install_db --user=mysql --datadir=/data/mysqldata/


cp support-files/mysql.server /etc/rc.d/init.d/mysqld
ldconfig

echo "PATH=$PATH:/usr/local/mysql/bin" > /etc/profile.d/mysql.sh
source /etc/profile.d/mysql.sh

chkconfig mysqld on
service mysqld start

./bin/mysqladmin -u root password 'Dicosp@ss123!'

# 授权远程登录
show grants for root;

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Dicosp@ss123!' WITH GRANT OPTION;
flush privileges;



# 查看日志信息
mysql> show global variables like '%log%';

# 查看主节点二进制日志列表
mysql> show master logs;

# 查看主节点server id
mysql> show global variables like '%server%';


# 主从同步用户-主执行
2）授权从库登录
a.创建同步用户，
create user repl identified by 'Syk^ziuOOcsLCaw9';

##新建远程访问用户
create user kzt_zt identified by 'Syk^ziuOOcsLCaw9';
GRANT ALL PRIVILEGES ON *.* TO 'kzt_zt'@'%' IDENTIFIED BY 'Syk^ziuOOcsLCaw9' WITH GRANT OPTION;
flush privileges;
b.从库授权,授权从库IP
grant replication slave on *.* to repl@'10.225.72.198' identified by 'Syk^ziuOOcsLCaw9';
c.查看主库状态
show master status;
d.同步主库-从上执行，主库IP
change master to master_host='10.225.72.143',master_user='repl',master_password='Syk^ziuOOcsLCaw9',master_log_file='mysql-bin.000003',master_log_pos=1037;
start slave;
show slave status\G;
```

```shell

```

```shell
# 从库 my.cnf文件
[client]
default-character-set=utf8mb4
socket=/usr/local/mysql/data/mysql.sock


[mysql]
default-character-set=utf8mb4
socket=/usr/local/mysql/data/mysql.sock
#wait_timeout=86400


[mysqld]
relay-log=relay-log
relay-log-index=relay-log.index
server-id=12
innodb_file_per_table=ON
skip_name_resolve=ON

datadir=/data/mysqldata
log-error=/data/mysqldata/mysql.log
socket=/usr/local/mysql/data/mysql.sock
symbolic-links=0
skip-name-resolve

port = 3306
basedir=/usr/local/mysql
max_connections=2500

character-set-client-handshake=FALSE
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'

default-storage-engine=INNODB
lower_case_table_name=1
max_allowed_packet=16M
interactive_timeout=600
wait_timeout=600

```





wget https://mirrors.163.com/mysql/Downloads/MySQL-5.7/mysql-5.7.37-el7-x86_64.tar.gz

