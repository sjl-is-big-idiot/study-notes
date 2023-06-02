#####################################################

##                     安装gitlab                  ##

#####################################################
GitLab官方文档 https://docs.gitlab.com/ee/

[root@VM-1-8-centos module]# yum install curl policycoreutils openssh-server openssh-clients postfix  安装依赖性
[root@VM-1-8-centos module]# systemctl status postfix.service  查看服务状态确保开启
[root@VM-1-8-centos module]# yum install -y net-tools  安装工具包
[root@VM-1-8-centos module]# yum install policycoreutils-python
[root@VM-1-8-centos module]# ls
[root@VM-1-8-centos module]# gitlab-ce-13.12.7-ce.0.el7.rpm
[root@VM-1-8-centos module]# rpm -ivh gitlab-ce-13.12.7-ce.0.el7.rpm  安装gitlab服务


       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.



     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/


[root@VM-1-8-centos module]# gitlab-
gitlab-backup     gitlab-ctl        gitlab-psql       gitlab-rails      gitlab-rake       gitlab-redis-cli

# gitlab配置文件
[root@VM-1-8-centos module]# vim /etc/gitlab/gitlab.rb

external_url 'http://82.69.224.232'

[root@VM-1-8-centos module]# gitlab-ctl reconfigure

[root@VM-1-8-centos module]# gitlab-ctl restart
ok: run: alertmanager: (pid 22762) 1s
ok: run: gitaly: (pid 22773) 0s
ok: run: gitlab-exporter: (pid 22804) 0s
ok: run: gitlab-workhorse: (pid 22806) 0s
ok: run: grafana: (pid 22815) 1s
ok: run: logrotate: (pid 22828) 0s
ok: run: nginx: (pid 22922) 1s
ok: run: node-exporter: (pid 22930) 0s
ok: run: postgres-exporter: (pid 22941) 1s
ok: run: postgresql: (pid 22952) 0s
ok: run: prometheus: (pid 22961) 0s
ok: run: puma: (pid 22973) 1s
ok: run: redis: (pid 22979) 0s
ok: run: redis-exporter: (pid 22985) 1s
ok: run: sidekiq: (pid 22994) 0s



浏览器访问 http://81.69.224.232 报502
查看gitlab日志 /var/log/gitlab/nginx/gitlab_access.log

网上说是端口占用，
修改unicorn的端口为9090，提示unicorn在gitlab13之后弃用了，使用puma代替，修改puma端口为9090，ps -ef 发现Prometheus用的9090，修改puma端口为8081，发现并不是这个原因导致的502

但是啥也没做，隔一会gitlab正常了。


密码 root/gitlab123




#####################################################
##                     gitlab功能                  ##
#####################################################

管理：获得对您的业务表现的可见性和洞察力。
计划：无论您的流程如何，GitLab 都提供了强大的规划工具来让每个人保持同步。
gitlab导入Github上的仓库，实际是通过personal access token做认证，然后使用如下命令克隆了仓库到gitlab中。
git clone --bare https://*****@github.com/sjl-is-big-idiot/bootstrap-4-login-page.git


#####################################################
##                     gitlab操作                  ##
#####################################################

 配置并启动gitlab-ce
 gitlab-ctl reconfigure
 ##时间可能比较长,请你耐心等待即可!~~~
 关闭：
 gitlab-ctl stop
 启动：
 gitlab-ctl start
 重启：
 gitlab-ctl restart


 Set up a specific runner manually
Install GitLab Runner and ensure it's running.
Register the runner with this URL:
http://81.69.224.232/ 

And this registration token:
L5xn-Ufu8WXmsXKFQuKW 

#####################################################
##                  安装gitlab runner              ##
#####################################################

 https://docs.gitlab.com/runner/install/linux-repository.html

# gitlab-runner的配置文件路径
/etc/gitlab-runner/config.tom

# 注册gitlab-runner
[root@VM-1-8-centos module]# sudo gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=10236 revision=4b9e985a version=14.4.0
Running in system-mode.                            
                                                   
Enter the GitLab instance URL (for example, https://gitlab.com/):
http://81.69.224.232/
Enter the registration token:
L5xn-Ufu8WXmsXKFQuKW
Enter a description for the runner:
[VM-1-8-centos]: firts runner
Enter tags for the runner (comma-separated):
v1.0
ERROR: Registering runner... failed                 runner=L5xn-Ufu status=couldn't execute POST against http://81.69.224.232/api/v4/runners: Post http://81.69.224.232/api/v4/runners: dial tcp 81.69.224.232:80: i/o timeout
PANIC: Failed to register the runner. You may be having network problems. 

# 注册失败，分析原因是设置了81.69.224.232的80端口安全组，本机不能访问这个公网IP的80端口。
# 在/etc/hosts中添加 127.0.0.1 81.69.224.232 仍然不行