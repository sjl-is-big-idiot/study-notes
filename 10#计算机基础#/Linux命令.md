

### Linux中各目录的作用

![img](Linux命令.assets/20181220143703134)

【常见目录说明】

| 目录        |                                                              |
| ----------- | ------------------------------------------------------------ |
| /bin        | 存放二进制可执行文件(ls,cat,mkdir等)，常用命令一般都在这里。 |
| /etc        | 存放系统管理和配置文件                                       |
| /home       | 存放所有用户文件的根目录，是用户主目录的基点，比如用户user的主目录就是/home/user，可以用~user表示 |
| /usr        | 用于存放系统应用程序，比较重要的目录/usr/local 本地系统管理员软件安装目录（安装系统级的应用）。这是最庞大的目录，要用到的应用程序和文件几乎都在这个目录。/usr/x11r6 存放x window的目录/usr/bin 众多的应用程序 /usr/sbin 超级用户的一些管理程序 /usr/doc linux文档 /usr/include linux下开发和编译应用程序所需要的头文件 /usr/lib 常用的动态链接库和软件包的配置文件 /usr/man 帮助文档 /usr/src 源代码，linux内核的源代码就放在/usr/src/linux里 /usr/local/bin 本地增加的命令 /usr/local/lib 本地增加的库 |
| /opt        | 额外安装的可选应用程序包所放置的位置。一般情况下，我们可以把tomcat等都安装到这里。 |
| /proc       | 虚拟文件系统目录，是系统内存的映射。可直接访问这个目录来获取系统信息。 |
| /root       | 超级用户（系统管理员）的主目录（特权阶级^o^）                |
| /sbin       | 存放二进制可执行文件，只有root才能访问。这里存放的是系统管理员使用的系统级别的管理命令和程序。如ifconfig等。 |
| /dev        | 用于存放设备文件。                                           |
| /mnt        | 系统管理员安装临时文件系统的安装点，系统提供这个目录是让用户临时挂载其他的文件系统。 |
| /boot       | 存放用于系统引导时使用的各种文件                             |
| /lib        | 存放跟文件系统中的程序运行所需要的共享库及内核模块。共享库又叫动态链接共享库，作用类似windows里的.dll文件，存放了根文件系统程序运行所需的共享文件。 |
| /tmp        | 用于存放各种临时文件，是公用的临时文件存储点。               |
| /var        | 用于存放运行时需要改变数据的文件，也是某些大文件的溢出区，比方说各种服务的日志文件（系统启动日志等。）等。 |
| /lost+found | 这个目录平时是空的，系统非正常关机而留下“无家可归”的文件（windows下叫什么.chk）就在这里 |

###  查看文件最新的修改时间

```shell
[atguigu@hadoop102 ~]$ stat test.txt 
  File: ‘test.txt’
  Size: 612       	Blocks: 8          IO Block: 4096   regular file
Device: fd02h/64770d	Inode: 87          Links: 1
Access: (0775/-rwxrwxr-x)  Uid: ( 1001/ atguigu)   Gid: ( 1001/ atguigu)
Context: unconfined_u:object_r:home_bin_t:s0
Access: 2021-08-28 18:32:58.415382468 +0800
Modify: 2021-06-15 12:56:25.036031722 +0800
Change: 2021-06-15 12:56:25.039031722 +0800
 Birth: -
```



### 单用户模式修改密码

[常见 LInux 系统进入单用户模式](https://blog.csdn.net/gaofei0428/article/details/115323647)

以centos7为例，

开机在 grub 引导界面，在默认选项上按下 e 键进入编辑模式：

![20210330141115194](Linux命令.assets/20210330141115194.png)


找到 linux 这一行，在行末添加 rd.break（注意这里是一整行），使用 Ctrl + x 进入单用户模式：

![20210330141350236](Linux命令.assets/20210330141350236.png)

看到如下画面就证明成功进入单用户模式

![20210330141549915](Linux命令.assets/20210330141549915.png)

然后执行以下操作

![20210330142022942](Linux命令.assets/20210330142022942.png)


exit 退出后 reboot 系统

![2021033014211558](Linux命令.assets/2021033014211558.png)



### 29. linux 如何设置开机自启动程序

1. 最简单粗暴的方式直接在脚本`/etc/rc.d/rc.local`(和`/etc/rc.local`是同一个文件，软链)末尾添加自己的脚本；然后，增加脚本执行权限

2. `crontab -e`

   ```bash
   crontab -e
   @reboot /home/user/test.sh
   ```

3. 每次登录自动执行。可以设置每次登录自动执行脚本，在`/etc/profile.d/`目录下新建sh脚本， `/etc/profile`会遍历`/etc/profile.d/*.sh`

4. 在`/etc/rc[0-6].d/`目录建立软链接，软链接指向`/etc/init.d/`目录下的控制脚本

### 30. linux查看某个服务所用的端口是多少？

```bash
netstat –tunlp | grep <pid>
```

### 31. 查看某端口的占用情况

```bash
lsof –i:<port>
或者
netstat –tunlp | grep <port>
```

### Crontab定时任务的时间间隔是相对于什么时间而言？

我想说的是，定时任务如下：

```bash
*/3 * * * * echo `date` >> test_crontab.log
```

当前时间是 2020/10/14 19:49:51 ，那么请问上面的输出应该是什么样的呢？

相对于当前时间来间隔3秒吗？

```bash
2020/10/14 19:49:51
2020/10/14 19:52:51
2020/10/14 19:55:51
......
```

还是相对于00:00:00而言来间隔3秒？

```bash

```

实际是这样的

```bash
2020/10/14 19:51:00
2020/10/14 19:54:00
2020/10/14 19:57:00
```

结论：是相对于当前的整点时间而言的。

改成没7秒输出一次

```bash
2020/10/14 19:56:00
```

这也证实了我们的结论。

> crontab 对应的文件为/var/spool/crontab/下对应的用户。我们通过crontab -e修改的就是这个目录下面对应的文件。

### crontab时间与系统时间不一致

[解决crontab执行时间与系统时间不一致的问题](https://www.cnblogs.com/showker/p/12660167.html)

[What is /etc/timezone used for?](https://unix.stackexchange.com/questions/452559/what-is-etc-timezone-used-for)]

```shell
[root@prometheus ~]# cat /etc/localtime
TZif2�Y^��	�p�ӽ����|@�;>�Ӌ{��B���E"�L���<��fp����A|��R i�� ~��!I}�"g� #)_�$G� %|&'e &�^(G (�@~�p�CDTCSTTZif2
                                                     ����~6C)�����Y^������	�p�����ӽ������������|@�����;>�����Ӌ{������B�������E"�����L�������<������fp������������A|��R i�� ~��!I}�"g� #)_�$G� %|&'e &�^(G (�@q�~�pLMTCDTCST
CST-8
[root@prometheus ~]# 
```

使用`cat /etc/localtime`查看当前的系统时区，在出现此问题的服务器中发现，该文件没有内容，实际上是其软链接的文件没有内容。

我们都知道Linux中一切皆文件。实际上，`/etc/localtime`是一个软链接，它链接到`../usr/share/zoneifno/`下的某个文件，就使用这个文件所对应的时区作为系统时区。

```shell
[root@prometheus ~]# ll /etc/localtime
lrwxrwxrwx. 1 root root 35 Dec 14 20:42 /etc/localtime -> ../usr/share/zoneinfo/Asia/Shanghai
[root@prometheus ~]# ll /usr/share/zoneinfo/
Africa      Chile    GB         Indian       Mexico    posixrules  Universal
America     CST6CDT  GB-Eire    Iran         MST       PRC         US
Antarctica  Cuba     GMT        iso3166.tab  MST7MDT   PST8PDT     UTC
Arctic      EET      GMT0       Israel       Navajo    right       WET
Asia        Egypt    GMT-0      Jamaica      NZ        ROC         W-SU
Atlantic    Eire     GMT+0      Japan        NZ-CHAT   ROK         zone1970.tab
Australia   EST      Greenwich  Kwajalein    Pacific   Singapore   zone.tab
Brazil      EST5EDT  Hongkong   leapseconds  Poland    Turkey      Zulu
Canada      Etc      HST        Libya        Portugal  tzdata.zi
CET         Europe   Iceland    MET          posix     UCT
[root@prometheus ~]# ll /usr/share/zoneinfo/Asia
Aden       Calcutta     Hong_Kong     Kuching       Qostanay       Thimphu
Almaty     Chita        Hovd          Kuwait        Qyzylorda      Tokyo
Amman      Choibalsan   Irkutsk       Macao         Rangoon        Tomsk
Anadyr     Chongqing    Istanbul      Macau         Riyadh         Ujung_Pandang
Aqtau      Chungking    Jakarta       Magadan       Saigon         Ulaanbaatar
Aqtobe     Colombo      Jayapura      Makassar      Sakhalin       Ulan_Bator
Ashgabat   Dacca        Jerusalem     Manila        Samarkand      Urumqi
Ashkhabad  Damascus     Kabul         Muscat        Seoul          Ust-Nera
Atyrau     Dhaka        Kamchatka     Nicosia       Shanghai       Vientiane
Baghdad    Dili         Karachi       Novokuznetsk  Singapore      Vladivostok
Bahrain    Dubai        Kashgar       Novosibirsk   Srednekolymsk  Yakutsk
Baku       Dushanbe     Kathmandu     Omsk          Taipei         Yangon
Bangkok    Famagusta    Katmandu      Oral          Tashkent       Yekaterinburg
Barnaul    Gaza         Khandyga      Phnom_Penh    Tbilisi        Yerevan
Beirut     Harbin       Kolkata       Pontianak     Tehran
Bishkek    Hebron       Krasnoyarsk   Pyongyang     Tel_Aviv
Brunei     Ho_Chi_Minh  Kuala_Lumpur  Qatar         Thimbu
```

所以，针对我所遇到的问题，我就从其他正常的服务器中拷贝了一个正常的`/usr/share/zoneinfo/Asia/Shanghai`文件覆盖当前服务器的对应路径的文件。然后，重启`crond`服务即可。

```shell
service crond restart 
```

查看`/etc/localtime`是否正常了。

###  ssh配置免密登录

![wKioL1naFYGxqnPsAACTluQeBiY673](Linux命令.assets/wKioL1naFYGxqnPsAACTluQeBiY673.png)

[SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)

[Shell脚本实现ssh免密登录及批量配置管理](https://blog.51cto.com/vinsent/1970780)



> SSH 是一种网络协议，用于计算机之间的加密登录。
>
> SSH 只是一种协议，存在多种实现，既有商业实现，也有开源实现。例如，常用的OpenSSH。
>
> SSH之所以能够保证安全，是因为它采用了公钥加密。
>
> 1. 远程主机收到用户的登录请求，把自己的公钥发送给用户。
> 2. 用户使用这个公钥，将登录密码加密后，发送回来。
> 3. 远程主机使用自己的私钥，解密登录密码，如果密码正确，就同意用户登录。

免密登录的原理：

> 用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机使用事先储存的公钥进行解密，如果解密成功，就证明用户是可信的，直接允许登录shell，不再要求密码。

[SSH 三步解决免密登录](https://blog.csdn.net/jeikerxiao/article/details/84105529?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3.control)

1. 检查现有的ssh密钥对

   ```shell
   ls -al ~/.ssh/id_*.pub
   ```

   如果存在现有密钥，则跳过第二步。

2. 生成新的ssh密钥对

   ```shell
   # -t表示使用的加密算法
   # -b表示生成的密钥有多少位
   # -C表示注释
   ssh-keygen -t rsa -b 4096 -C yourEmail
   ```

   连续回车，就可以生成新的ssh密钥对了。

   ![ssh-keygen](Linux命令.assets/ssh-keygen.png)

   查看`~/.ssh`目录

   ![ssh-id_rsa](Linux命令.assets/ssh-id_rsa.png)

   `~/.ssh/known_hosts`中是已经保存过的远程主机的公钥，表示这些公钥是可以信赖的。

3. 将公钥复制到其他主机

   ```shell
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.135.102
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.135.103
   ```

   上面的命令，将此主机的ssh公钥上传到了192.168.135.102，192.168.135.103的`~/.ssh`目录中的`authorized_keys`文件中了。可以查看该文件内容，进行验证。

4. 测试免密登录

   ```shell
   ssh 192.168.135.102
   ```



补充，如果有多个主机需要配置ssh免密登录，那么可以使用shell脚本来实现。如：

```shell
#!/bin/bash

for ip in ``;
do
	ssh-copy-id -i ~/.ssh/id_rsa.pub root@$ip
done
```







### ssh 远程登录主机并执行命令

```shell
ssh root@ip "your command"
# 例子，远程登录某主机，并查看该主机根目录磁盘空间使用情况。
ssh root@192.168.0.3 "df -h |grep -w /"
```

### 查看系统版本信息

```bash
uname -a
lsb_release -a
cat /etc/issue
<<<<<<< HEAD
cat /prov/version
=======
>>>>>>> 3777eaa39c5bdf7bdd5ad246caaca1cf3fe248ab
cat /etc/redhat-release
```

### 用户和用户组相关

#### 创建用户

```bash
# 增加用户
useradd -U -d /home/userName -m -p 123456 userName

adduser userName
# 增加用户组
groupadd groupName
# 指定组的方式增加用户
useradd -g groupName userName
# 显示用户所属的用户组
groups
# 查看所有group
cat /etc/group
# 查看所有用户
cat /etc/passwd
```

#### 修改用户密码

```bash
# 修改root用户的密码
passwd
# 修改其他用户的密码
passwd <username>
#
echo 'xxx' |passwd testuser
```
#### 删除用户

```bash
userdel -r userName
groupdel -r groupName
```

#### 修改用户权限

```shell
# 为用户设置root权限
# 如想将新增的用户设为root权限，可通过+ sudo 方式执行系统命令操作。
# 并不是任何用户都可以使用sudo的，需要root账号给与设置。
1. vim /etc/sudoers
分别在
root	ALL=(ALL)	ALL
和
%wheel 	ALL=(ALL)	ALL
下添加可有使用sudo的用户的和用户组。
```

### 文件权限相关

#### 修改文件权限

```shell
chmod 642 filename
```

#### 修改文件所属用户

```shell
chown -R userName:groupName filename
```

#### root用户且有读写权限，但是仍然无法修改文件

```shell
# 取消文件的`immutable`
chmod -i 文件路径
# 让该文件变为`immutable`，不可变的
chmod +i 文件路径
```



### 查看文件md5和sha56值

`md5sum`用于生成和校验文件的md5值。它会对文件内容进行校验，而与文件名无关。有很小的概率不同文件生成的md5值相同。

常用场景，在网络传输时，如`wget` 下载远程文件，下载完成后校验该文件的md5值，看源文件的md5和下载到本地的md5值是否相同，来判断该文件的传输是否无异常。

**Linux系统**

```shell
命令格式
md5sum [OPTION]... [FILE]...
命令选项
-b或 --binary:以二进制模式读入文件；
-t或 --text:以文本文件模式读入文件（默认）；
-c或 --check:用来从文件中读取md5信息检查文件的一致性；
--status:该选项与check一起使用，在check时不输出，根据返回值表示检查结果；
-w或 --warn:在check时，检查输入的md5信息有没有非法行，若有则输出相应信息


sha256su  文件路径
```



**Windows系统**

```cmd
certutil -hashfile 文件路径 MD5

certutil -hashfile 文件路径 SHA1

certutil -hashfile 文件路径 SHA256
```

