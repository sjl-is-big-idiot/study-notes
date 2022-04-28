# 1. 介绍

# 安装memcached

```shell

```



# 2. 启动memcached

```shell
memcached -u root -p 11211 -m 2048 -c 2048 -d
```

> -u表示用户为root，
>
> -m表示最大内存为2048M，
>
> -p表示监听端口为11211，
>
> -c表示最大连接数为2048，
>
> -d表示作为守护进程运行。

```shell
-p 监听的端口
-l 连接的IP地址, 默认是本机
-d start 启动memcached服务
-d restart 重起memcached服务
-d stop|shutdown 关闭正在运行的memcached服务
-d install 安装memcached服务
-d uninstall 卸载memcached服务
-u 以的身份运行 (仅在以root运行的时候有效)
-m 最大内存使用，单位MB。默认64MB
-M 内存耗尽时返回错误，而不是删除项
-c 最大同时连接数，默认是1024
-f 块大小增长因子，默认是1.25-n 最小分配空间，key+value+flags默认是48
-h 显示帮助
```



3. 

# 4. memcached常用命令

```shell
增: add 往内存增加一行新记录

语法:add key flag expire length

key  给值起一个独特的名字
flag  标志,要求为一个正整数
expire 有效期  length 缓存的长度(字节为单位)

flag的意义:
memcached基本文本协议,传输的东西,理解成字符串来存储.
想:让你存一个php对象,和一个php数组,怎么办?
答:序列化成字符串,往出取的时候,自然还要反序列化成对象/数组/json格式等等.
这时候,flag的意义就体现出来了.
比如,1就是字符串,2反转成数组3,反序列化对象.....
expire的意义:
设置缓存的有效期,有3种格式
1:设置秒数, 从设定开始数,第n秒后失效.
2:时间戳, 到指定的时间戳后失效.
比如在团购网站,缓存的某团到中午12:00失效.add key 0 13792099996
3:设为0.不自动失效.
注: 有种误会,设为0,永久有效.错误的.

1:编译memcached时,指定一个最长常量,默认是30天.

所以,即使设为0,30天后也会失效.
2:可能等不到30天,就会被新数据挤出去.

delete 删除
delete key [time seconds]
删除指定的key.如加可选参数time,则指删除key,并在删除key后的time秒内,不允许
get,add,replace操作此key.


replace 替换
replace key flag expire length
参数和add完全一样,不单独写


get查询
get key
返回key的值
```

| Command                                  | Description                                                  | Example                    |
| ---------------------------------------- | ------------------------------------------------------------ | -------------------------- |
| get                                      | Reads a value                                                | get mykey                  |
| set                                      | Set a key unconditionally                                    | set mykey 0 60 5           |
| add                                      | Add a new key                                                | add newkey 0 60 5          |
| replace                                  | Overwrite existing key                                       | replace key 0 60 5         |
| append                                   | Append data to existing key                                  | append key 0 60 15         |
| prepend                                  | Prepend data to existing key                                 | prepend key 0 60 15        |
| incr                                     | Increments numerical key value by given number               | incr mykey 2               |
| decr                                     | Decrements numerical key value by given number               | decr mykey 5               |
| delete                                   | Deletes an existing key                                      | delete mykey               |
| flush_all                                | 在指定time秒后，清除缓存中的键值对，直接flush_all则是立即清除 | flush_all [time] [noreply] |
| Invalidate all items in n seconds        | flush_all 900                                                |                            |
| stats                                    | Prints general statistics                                    | stats                      |
| Prints memory statistics                 | stats slabs                                                  |                            |
| Prints memory statistics                 | stats malloc                                                 |                            |
| Print higher level allocation statistics | stats items                                                  |                            |
|                                          | stats detail                                                 |                            |
|                                          | stats sizes                                                  |                            |
| Resets statistics                        | stats reset                                                  |                            |
| version                                  | Prints server version.                                       | version                    |
| verbosity                                | Increases log level                                          | verbosity                  |
| quit                                     | Terminate telnet session                                     | quit                       |



查看memcached版本,telnet连接memcached，stats查看版本信息

```shell
telnet ip 11211
stats

STAT pid 22443 #Memcached服务器进程ID
STAT uptime 17543398 #服务器已运行秒数
STAT time 1438505866	#服务器当前Unix时间戳
STAT version 1.4.22	#Memcached版本
STAT libevent 2.0.20-stable	#
STAT pointer_size 64	#操作系统指针大小
STAT rusage_user 63.916283	#进程累计用户时间
STAT rusage_system 36.312479	#进程累计系统时间
STAT curr_connections 5	#当前连接数量
STAT total_connections 36	#Memcached运行以来连接总数
STAT connection_structures 7	#Memcached分配的连接结构数量
STAT reserved_fds 20	#内部使用的miscfds 数量
STAT cmd_get 15	#get命令请求次数
STAT cmd_set 6	#set命令请求次数
STAT cmd_flush 1	#flush命令请求次数
STAT cmd_touch 0	#touch命令请求次数
STAT get_hits 9	#get命令命中次数
STAT get_misses 6	#get命令未命中次数
STAT delete_misses 0	#delete命令未命中次数
STAT delete_hits 1	#delete命令命中次数
STAT incr_misses 0	#incr命令未命中次数
STAT incr_hits 0	#incr命令命中次数
STAT decr_misses 0	#decr命令未命中次数
STAT decr_hits 0	#decr命令命中次数
STAT cas_misses 0	#cas命令未命中次数
STAT cas_hits 0	#cas命令命中次数
STAT cas_badval 0	#使用擦拭次数
STAT touch_hits 0	#touch命中次数
STAT touch_misses 0	#touch失败次数
STAT auth_cmds 0	#认证次数（包括成功和失败）
STAT auth_errors 0	#认证失败次数
STAT bytes_read 5815	#读取总字节数
STAT bytes_written 2108	#总写入数量数
STAT limit_maxbytes 67108864	#分配的内存总大小（字节）
STAT accepting_conns 1	#服务器是否达到过最大连接（0/1）
STAT listen_disabled_num 0	#失效的监听数
STAT threads 4	#当前线程数
STAT conn_yields 0	#连接操作主动放弃数目
STAT hash_power_level 16	#
STAT hash_bytes 524288	#当前使用的Hash table容量大小
STAT hash_is_expanding 0	#指定Hash table是否自动增长
STAT malloc_fails 0	#malloc内存分配失败的次数
STAT bytes 74	#当前存储占用的字节数
STAT curr_items 1	#当前缓存 item 数量
STAT total_items 5	#从服务启动后，总的存储缓存 i tem 数量
STAT expired_unfetched 0
STAT evicted_unfetched 0
STAT evictions 0	#驱逐了多少次key
STAT reclaimed 1	#已过期的数据条目来存储新数据的数目
STAT crawler_reclaimed 0
STAT lrutail_reflocked 0
```



语义：用于显示slab中item的数目和存储时长。

```shell
stats items
```

语义：用于查看slab的信息，包括chunk的大小、数据、使用情况等。

```shell
stats slabs
```

 语义：用于查看所有item的大小和数量。

  注意：Memcached 1.4.27+版本自动开启了stats sizes命令，1.4.27之前的版本需要启动时指定-o track_sizes参数来开启。

```shell
stats sizes
```



# 参考文献

[Memcached 统计命令详解及应用](https://blog.csdn.net/securitit/article/details/109356239)