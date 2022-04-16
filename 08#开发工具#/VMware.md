### 6.1 VMware文件过大，导致C盘过大

VMware的虚拟机文件默认存放在`C:\Users\sjl\Documents\Virtual Machines\`目录下，只需要将下面的文件放到`D:\Virtual Machines\`目录下，然后在 VMware`文件 > 打开 > xxx.vmx` 即可。

### 6.2 VMware虚拟机和宿主机可以相互ping通，但是宿主机无法访问虚拟机中开启的服务

宿主机 -> 虚拟机 能ping通

虚拟机 -> 宿主机 能ping通

这说明ip是没有问题的，可能是因为没有开放端口导致的服务无法访问。

1. 检查windows10的防火墙是否开启，需要关闭。检查发现已经关闭了。

2. 检查虚拟机的防火墙是否开启。发现虚拟机防火墙已经关闭了。

   ```shell
   # 查看firewalld状态， 结果显示inactive，未开启。
   systemctl status firewalld
   # 查看iptables状态
   systemctl status iptables
   Unit iptables.service could not be found.
   ```

3. 启动虚拟机的firewalld，并开放指定端口。

   ```shell
   systemctl start firewalld
   systemctl status firewalld
   
   # 添加防火墙规则
   firewall-cmd --zone=public --list-ports
   firewall-cmd --zone=public --add-port=7474/tcp --permanent
   # 每次更改firewall规则后，需要重新加载，获取重启firewalld才能生效
   firewall-cmd --zone=public --reload
   或
   sytemctl restart firewalld
   
   # 其他命令
   # 查看版本： 
   firewall-cmd --version
   
   # 查看帮助： 
   firewall-cmd --help
   
   # 显示状态： 
   firewall-cmd --state
   
   # 查看所有打开的端口： 
   firewall-cmd--zone=public --list-ports
   
   # 查看所有配置的规则
   firewall-cmd --list-all
   
   # 更新防火墙规则： 
   firewall-cmd --reload
   
   # 查看区域信息:  
   firewall-cmd--get-active-zones
   
   # 查看指定接口所属区域： 
   firewall-cmd--get-zone-of-interface=eth0
   
   # 拒绝所有包：
   firewall-cmd --panic-on
   
   # 取消拒绝状态： 
   firewall-cmd --panic-off
   
   # 查看是否拒绝： 
   firewall-cmd --query-panic
   
   
   # 添加,（--permanent永久生效，没有此参数重启后失效）
   firewall-cmd --zone=public --add-port=80/tcp --permanent
   # 对指定ip，开放指定端口
   firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="x.x.x.x" port protocol="tcp" port="7474" accept"
   
   # 重新载入
   firewall-cmd --reload
   
   # 查看
   firewall-cmd --zone=public --query-port=80/tcp
   
   # 删除开发的端口
   firewall-cmd --zone=public --remove-port=80/tcp --permanent
   
   # 删除 rich-rule
   firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="x.x.x.x" port protocol="tcp" port="7474" accept"
   ```

   

![image-20210407174746112](08#开发工具#.assets/image-20210407174746112.png)



> 至于为什么虚拟机的防火墙都没开，却不能访问其端口，这个还不清楚。
