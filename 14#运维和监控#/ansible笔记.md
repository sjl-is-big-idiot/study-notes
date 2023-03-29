ansible

1. 安装
	yum install python3-pip
	# 从官方下载比较慢，我们使用清华的镜像源下载
	pip3 install --user -i https://pypi.tuna.tsinghua.edu.cn/simple ansible
	
	pip3 install --user -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade setuptools
	pip3 install --user -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip
	
	不要使用pip3 install --user，这样会在当前用户，如hadoop用户，下安装包，安装的位置为 /home/hadoop/.local/bin, /home/hadoop/.local/lib。然后其他用户是无法使用此种方式安装的包的。
	要卸载可以通过 pip3 freeze 列出所有的包
	pip3 uninstall 包1 包2 ...
	
	pip3 uninstall -y ansible ansible-core cffi cryptography Jinja2 MarkupSafe packaging pycparser pyparsing PyYAML resolvelib
	python3 -m pip uninstall pip
	root用户下执行，
	
	pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple ansible
	
	pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade setuptools
	pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip
	
	
	还是这两句好使
	yum install epel-release
	yum install ansible

2. 配置
vim ~/ansible_workers

[workers]
hadoop322-node01
hadoop322-node02
hadoop322-node03
3. 使用

ansible workers -i ansible_hosts -m shell -a 'mkdir /opt/modules/apache-zookeeper-3.6.1-bin/zkData'
4. 问题





