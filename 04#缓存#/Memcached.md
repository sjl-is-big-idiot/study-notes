# 1. 介绍

# 2. 安装memcached

memcached依赖libevent

```shell
yum install memcached
```



# 3. 启停memcached

```shell
/usr/local/memcached -u root -p 11211 -m 2048 -c 2048 -d
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



# 4. memcached原理

本节下面内容参考自[Memcached深入分析及内存调优](https://www.ktanx.com/blog/p/2248)，这篇文章讲得很仔细。

`memcached`内存分配如下图所示：

![QQ图片20150427203745](Memcached.assets/dolX56enSx3XLebD.png)

> `Memcached`在分配内存时是以`Page`为单位的，默认情况下一个`Page`是1M，内部是一个个`chunk`，当`chunk`的大小等于Page大小时也就是`Memcached`所能存储的最大数据大小了，可以在启动时通过-l来指定它，最大可以支持128M。
>
> `Memcached`并不是将所有不同大小的数据都存放在一起的，而是将内存空间划分为一个个的`slab`，每个`slab`只负责一定范围内的数据。上图中，slab1只负责1 - 96bytes的数据，slab2负责9 - 120bytes的数据。
>
> 在存储数据时，如果这个`item`对应的`slab`还没有创建则申请一个`page`的内存，将这个`page`按照所在`slab`中`chunk`的大小进行分割，然后将`item`存入。
>
> 如果已经创建存在了，判断对应的`slab`是否用完，没用完直接存储。
>
> 如果对应的`slab`已经用完了，看内存是否用完，没用完会申请一个新的`page`进行分割存储，用完了则直接进行`LRU`(`Least Recent Use`，最近最少使用，当触发驱逐key时优先删除最近最少使用的key)。
>
> </br>
>
> `Memcached`通过指定的成长因子(`-f`指定，默认1.25倍)来决定每个`slab`中`chunk`增长的范围，第一个`slab`的大小可以通过`-n`来设定。
>
> 当数据进来时`Memcached`会选择一个大于等于最接近的`slab`来进行存储。例如当`item`大小为95时将存储到`chunk`为96bytes的slab1，`item`大小为97时则会存储到`chunk`大小为120的slab2.
>
> 这样分配的好处是速度快，避免大量重复的初始化和清理操作，有效的避免了内存碎片的问题，但内存利用率上会有所浪费。
>
> <font style="color:red;">另外`Memcached`是**懒检测机制**，当存储在内存中的对象过期甚至是`flush_all`时，它并不会做检查或删除操作，只有在`get`时才检查数据对象是否应该删除。</font>
>
> <font style="color:red;">删除数据时，`Memcached`同样是**懒删除机制**，只在对应的数据对象上做删除标识并不回收内存，在下次分配时直接覆盖使用。</font>
>
> 了解了`Memcached`的内存分配机制，如何进行调优是不是自然而然的就明白了？
>
> 应该尽量的根据实际情况来设定`slab`的`chunk`的初始大小和增长因子，尽量减少内存的浪费。在某些情况下数据的长度都会集中在一个区域，如session。甚至会有定长的情况，如数据统计等。
>
> 还有一个重要调优的地方就是提高缓存命中率了，这个没有固定的方法，还得具体场景做具体业务分析，需要注意的就是，`Memcached`中`LRU`的操作是基于slab而非全局，分析时最好考虑这一点，这也就是有时候内存还没用完但数据却被回收了的原因。

# 5. memcached常用命令

连接`memcached`，端口通常默认为11211

```shell
telnet ip port
```

| Command                           | Description                                                  | Example                    |
| --------------------------------- | ------------------------------------------------------------ | -------------------------- |
| get                               | 返回Key对应的Value值                                         | get mykey                  |
| set                               | 无条件地设置一个Key值，没有就增加，有就覆盖，操作成功提示STORED | set mykey 0 60 5           |
| add                               | 添加一个Key值，没有则添加成功并提示STORED，有则失败并提示NOT_STORED | add newkey 0 60 5          |
| replace                           | 按照相应的Key值替换数据，如果Key值不存在则会操作失败         | replace key 0 60 5         |
| append                            | Append data to existing key                                  | append key 0 60 15         |
| prepend                           | Prepend data to existing key                                 | prepend key 0 60 15        |
| incr                              | Increments numerical key value by given number               | incr mykey 2               |
| decr                              | Decrements numerical key value by given number               | decr mykey 5               |
| delete                            | Deletes an existing key                                      | delete mykey               |
| flush_all                         | 在指定time秒后，清除缓存中的键值对，直接flush_all则是立即清除。flush_all删除后通过stats查看会看到item数量并没有减少。因为flush_all 实际上没有立即释放item所占用的内存 而是在随后陆续有新的item被储存时执行（这是由memcached的懒惰检测和删除机制决定的） | flush_all [time] [noreply] |
| Invalidate all items in n seconds | flush_all 900                                                |                            |
| stats                             | 返回MemCache通用统计信息（下面有详细解读）                   | stats                      |
| stats items                       | 返回各个slab中item的数目和最老的item的年龄（最后一次访问距离现在的秒数） |                            |
| stats slabs                       | 返回MemCache运行期间创建的每个slab的信息（下面有详细解读）   |                            |
| stats cachedump slab_id limit_num | 显示某个slab中的前limit_num个key列表                         |                            |
| stats detail [on 、off、dump]     | 设置或者显示详细操作记录                                     |                            |
| stats malloc                      | 输出内存统计信息                                             | stats malloc               |
|                                   | stats sizes输出两列，第一列是item的大小，第二列是item的个数。 | stats sizes                |
| stats reset                       | 重新统计数据，重新开始统计                                   | stats reset                |
| version                           | 返回当前MemCache版本号                                       | version                    |
| verbosity                         | Increases log level                                          | verbosity                  |
| quit                              | 关闭连接                                                     | quit                       |

上述命令示例：

1. 创建数据：

   ```shell
   set key2 0 30 2  # 创建一个可以存两个字节的key
   12  # 这是上面这个key的value
   STORED  # 存储成功
   set key1 0 30 3   # 创建一个可以存三个字节的key
   abc   # 这是上面这个key的value
   STORED   # 存储成功
   ```

   对set key2 0 30 2的解释：

   - set： 创建数据命令（command name）
   - key2： 创建一个名为key2的key （key）
   - 0 ： 特殊标记位（flags）
   - 30 ： 定义这个数据的过期时间为30秒（exptime）
   - 2 ： 定义这个key所能够存储的value长度，单位是字节 （bytes）

2. 查询数据：

   ```shell
   get key2   # 查询key2的数据
   VALUE key2 0 2   # key2的信息
   12   # key2的value
   END  # 查询结束
   
   get key1    # 查询key1的数据
   VALUE key1 0 3   # key1的信息
   abc		# key1的value
   END		# 查询结束
   ```

   数据过期后就查询不到了：

   ```shell
   get key2
   END
   get key1
   END
   ```

   

3. replace替换数据示例：

   ```shell
   set key1 1 100 4        
   1234
   STORED
   
   get key1
   VALUE key1 1 4
   1234
   END
   
   replace key1 1 200 5
   12345
   STORED
   
   get key1
   VALUE key1 1 5
   12345
   END
   ```

   

4. delete删除数据示例：

   ```shell
   set key1 0 200 5
   12345
   STORED
   
   get key1
   VALUE key1 0 5
   12345
   END
   
   delete key1
   DELETED
   
   get key1
   END
   ```

   

5. add添加数据示例：

   ```shell
   add key2 0 50 5
   12345
   STORED
   
   get key2
   VALUE key2 0 5
   12345
   END
   ```

   



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



# 6. 参考文献

[Memcached深入分析及内存调优](https://www.ktanx.com/blog/p/2248)

[Memcached入门教程](http://c.biancheng.net/view/6574.html)

[Memcached 统计命令详解及应用](https://blog.csdn.net/securitit/article/details/109356239)

[memcached的一些简单使用](https://blog.51cto.com/zero01/2055498)