#安装git

不是必须

```shell
yum -y install git 
```

#git下载仓库，或者zip下载

二选一即可

```shell
git clone https://github.com/hortonworks/hive-testbench.git
```

#安装maven

若没有安装maven，build时会自动安装maven

```shell
yum -y install maven
```

#build，compile tpcds

```shell
chown hadoop.haoop hive-testbench-hdp3.zip
su hadoop 
unzip hive-testbench-hdp3.zip
cd hive-hive-testbench-hdp3
./tpcds-build.sh 
```

#调整 tpcds-setup.sh的hiveserver2地址为真实的服务地址，比如 -n hadoop -u jdbc:hive2://x.x.x.x:7001

## 生成30G的数据
```shell
#前台执行
./tpcds-setup.sh 30
#挂后台执行
nohup runuser -l hadoop -c 'cd /home/hadoop/hive-testbench && FORMAT=parquet ./tpcds-setup.sh 10000' &


nohup FORMAT=parquet ./tpcds-setup.sh 10000 &
```