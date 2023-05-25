# 1. Superset概述

![image-20230512102133451](Superset.assets/image-20230512102133451.png)

## 1.1 什么是Superset

Apache Superset是一个现代数据探索和可视化平台。

Apache Superset是一个开源的、现代的、轻量级BI分析工具，能够对接多种数据源、拥有丰富的图标展示形式、支持自定义仪表盘，且拥有友好的用户界面，十分易用。

由于Superset能够对接常用的大数据分析工具，如Trino、Hive、Kylin、Druid等，且支持自定义仪表盘，故可作为数仓的可视化工具，应用于数据仓库的ADS！

截止到2023.05.12，Superset支持如下DB

![image-20230512103851994](Superset.assets/image-20230512103851994.png)

特性：

- 功能强大，易于使用
- Superset可以通过SQLAlchemy连接到任何基于SQL的数据源，包括PB级的现代云原生数据库和引擎。
- 现代化架构：轻量的、可高度扩展的、
- 丰富的可视化和仪表盘

支持：

- 可视化图表的方式分析数据
- 交互式仪表盘查看数据
- 写SQL查询来数据

## 1.2 Superset的目标



FAQ

- 一次可以join或query多个表吗？

  explore和view不支持多个表，可以是一个table或一个view，SQL lab支持多表join和query。

- 支持的数据源大小？

  Superset是在底层数据或数据引擎之上的thin layer，标准是在用户可接受的时间内完成查询并返回结果。对于分布式数据库而言TB级数据是ok。

- 如何创建可视化图表？

  参考：https://superset.apache.org/docs/contributing/creating-viz-plugins/

- 可以上传CSV并实现可视化吗？

  支持。

- 为什么query显示time out？

  Superset有超时时间的配置。

- 为什么map在geospatial可视化中不可见？

  需要先注册 [Mapbox.com](https://www.mapbox.com/)账户，获取到API key，及相关配置才行。

- 如何限制仪表盘的定时刷新时间间隔？

  dashboard JSON元数据字段中`timed_refresh_immune_slices`可以用来设置哪些slice不用定期自动刷新。

- 如果表的schema发生了变化怎么办？

  要让Superset发现您的新列，您所要做的就是转到“数据”->“数据集”，单击架构已更改的数据集旁边的编辑图标，然后从“列”选项卡中单击“从源同步列”。在场景后面，新列将被合并。

- Superset的后端数据库引擎可以支持哪些？

  Supserset使用的后端数据库是OLTP数据库，用于存储user、slice、dashboard等信息。经过测试的Superset的后端数据库有MySQL、PostgreSQL、SQLite。 Oracle、Microsoft SQL Server及其他OLTP数据库未经过测试。列式存储、非OLTP的数据（像Vertica、Redshift、Presto）这类数据库不能作为Superset的后端数据库，完全不能工作。

  几乎所有具有SqlAlchemy集成的数据库（并未限于OLTP才行）都可以作为Superset的数据源，注意是数据源而不是后端数据库。

- 如何配置OAuth的认证和授权？

- 是否可以强制使用某种配色？

  dashboard的JSON元数据中的`label_colors`配置可以设置label对应的颜色。

- Superset支持[insert database engine here]吗？

  “连接到数据库”部分提供了受支持数据库的最佳概述。该页面上未列出的数据库引擎也可以工作。我们依靠社区为这个知识库做出贡献。

- Superset提供公共的API吗？

  支持。

- 在edit Dataset view中的“小时偏移”是做什么的？

  在edit Dataset view中，可以指定时间偏移量。用于在时间列+或者-去指定的小时数。示例用途：用于将UTC时间转换为当地时间。



## 1.3 Superset支持的框架

## 1.4 Superset的架构





## 1.5 Superset的工作原理



# 2. Superset的安装

## 2.1 环境准备



# 3. Superset使用

## Superset配置到presto+https的连接

**背景**：客户在腾讯云搭建了emr-hadoop集群其中使用到了presto，且为presto继承了ssl+ldap。现在客户要使用superset作为BI分析软件。

**需求**：需要在superset中配置presto的连接，以做BI分析。

部署好superset之后，浏览器打开superset的地址。然后进入`Data` > `Databases`。

在Databases中，我们就可以创建superset想要连接的数据源了。

superset是使用python语言开发的。

在配置过程中可能会出现如下几种错误：

**旧版superset**

- Error：could not load database driver：presto

  ![image-20230525134651645](Superset.assets/image-20230525134651645.png)

  这是因为superset后台时通过sqlalchemy去连接每个数据源的，而要通过sqlalchemy访问presto需要使用到pyhive。需要先安装pyhive。

- (bultins.ValueError) Protocol must be https when passing a password [SQL: SHOW SCHEMAS]

  ![image-20230525134921373](Superset.assets/image-20230525134921373.png)

  在presto集成ldap时，则必须为presto使用https才能集成ldap，所以客户侧当时是使用自签名证书来实现的ssl。

  而在superset中配置presto的连接时，也得使用https才可以。

  解决办法：

  在配置presto连接的页面，在`Extra`栏填入如下内容：

  ```json
  {
      "metadata_params": {},
      "engine_params": {
        "connect_args": {
          "protocol": "https"
        }
      },
      "metadata_cache_timeout": {},
      "schemas_allowed_for_csv_upload": []
  }
  ```

  启动`"protocol":"https"`表示我们的连接使用的是https协议。这样就不会再报此错误了。

旧版superset的成功配置：

```bash
# 连接名
test1-presto
# 连接串，其中如果密码部分有@等特殊字符，可以使用 用urlparse对其进行编码之后的字符串代替特殊字符
# import urllib
# urllib.parse.quote('@')
# %40
# 表示如果密码中有@，则用%40代替@
presto://hadoop:hadoop@192.168.0.1:8443/hive/default

# Extra参数
{
    "metadata_params": {},
    "engine_params": {
      "connect_args": {
        "protocol": "https"
      }
    },
    "metadata_cache_timeout": {},
    "schemas_allowed_for_csv_upload": []
}
```



**新版superset**，的数据源连接配置在`Data` > `Databases` 。由于客户的presto+https+ldap。因此除了填写连接名、连接串之外，还需要在`Advanced`中找到`engine_args`栏，然后填入额外的连接参数。如下图所示：

![image-20230525135126067](Superset.assets/image-20230525135126067.png)

- Error：could not load database driver: PrestoEngineSpec

  ![image-20230525135055273](Superset.assets/image-20230525135055273.png)

  看前台的报错不明确，后台查看superset的日志发现如下报错：

  ```bash
  CommandException
  Traceback (most recent call last):
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/databases/commands/test_connection.py", line 81, in run
      engine = database.get_sqla_engine()
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/utils/memoized.py", line 50, in __call__
      value = self.func(*args, **kwargs)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/models/core.py", line 401, in get_sqla_engine
      raise self.db_engine_spec.get_dbapi_mapped_exception(ex)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/models/core.py", line 399, in get_sqla_engine
      return create_engine(sqlalchemy_url, **params)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/sqlalchemy/engine/__init__.py", line 525, in create_engine
      return strategy.create(*args, **kwargs)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/sqlalchemy/engine/strategies.py", line 61, in create
      entrypoint = u._get_entrypoint()
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/sqlalchemy/engine/url.py", line 172, in _get_entrypoint
      cls = registry.load(name)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/sqlalchemy/util/langhelpers.py", line 277, in load
      raise exc.NoSuchModuleError(
  sqlalchemy.exc.NoSuchModuleError: Can't load plugin: sqlalchemy.dialects:presto
  
  The above exception was the direct cause of the following exception:
  
  Traceback (most recent call last):
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/flask/app.py", line 1516, in full_dispatch_request
      rv = self.dispatch_request()
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/flask/app.py", line 1502, in dispatch_request
      return self.ensure_sync(self.view_functions[rule.endpoint])(**req.view_args)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/flask_appbuilder/security/decorators.py", line 89, in wraps
      return f(self, *args, **kwargs)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/views/base_api.py", line 113, in wraps
      raise ex
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/views/base_api.py", line 110, in wraps
      duration, response = time_function(f, self, *args, **kwargs)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/utils/core.py", line 1507, in time_function
      response = func(*args, **kwargs)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/utils/log.py", line 245, in wrapper
      value = f(*args, **kwargs)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/views/base_api.py", line 83, in wraps
      return f(self, *args, **kwargs)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/databases/api.py", line 709, in test_connection
      TestConnectionDatabaseCommand(g.user, item).run()
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/databases/commands/test_connection.py", line 133, in run
      raise DatabaseTestConnectionDriverError(
  superset.databases.commands.exceptions.DatabaseTestConnectionDriverError: Could not load database driver: PrestoEngineSpec
  ```

  哦，原来是客户重装了新版的superset之后，又没有装pyhive包。

  需要先确认客户是否使用了，虚拟环境安装superset。如果使用了虚拟环境，则需要先进入虚拟环境。

  ```bash
  # 假设superset运行在virtualenv中
  
  # 进入名为superset的虚拟环境
  source /root/Python-3.9.15/superset/bin/activate
  
  # 查看pip，是否为Python3.9的包管理工具
  pip -V
  
  # 查看是否有pyhive
  pip freeze |grep -i pyhive
  
  # 安装pyhive
  pip install pyhive
  
  # 重新启动superset
  export FLASK_APP=superset
  cd /root/Python-3.9.15/superset
  nohup bin/superset run -h 0.0.0.0 -p 8088 --with-threads --reload --debugger >/tmp/superset.log 2>&1 &
  ```

  重启superset之后，再次尝试仍报上述错误，后台查看superset的日志发现如下报错：

  ```bash
  superset.databases.commands.exceptions.DatabaseTestConnectionDriverError: Could not load database driver: PrestoEngineSpec
  2023-05-24 11:40:19,045:WARNING:superset.views.base:CommandException
  Traceback (most recent call last):
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/databases/commands/test_connection.py", line 81, in run
      engine = database.get_sqla_engine()
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/utils/memoized.py", line 50, in __call__
      value = self.func(*args, **kwargs)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/models/core.py", line 401, in get_sqla_engine
      raise self.db_engine_spec.get_dbapi_mapped_exception(ex)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/superset/models/core.py", line 399, in get_sqla_engine
      return create_engine(sqlalchemy_url, **params)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/sqlalchemy/engine/__init__.py", line 525, in create_engine
      return strategy.create(*args, **kwargs)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/sqlalchemy/engine/strategies.py", line 61, in create
      entrypoint = u._get_entrypoint()
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/sqlalchemy/engine/url.py", line 172, in _get_entrypoint
      cls = registry.load(name)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/sqlalchemy/util/langhelpers.py", line 275, in load
      return impl.load()
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/pkg_resources/__init__.py", line 2517, in load
      return self.resolve()
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/pkg_resources/__init__.py", line 2523, in resolve
      module = __import__(self.module_name, fromlist=['__name__'], level=0)
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/pyhive/sqlalchemy_presto.py", line 21, in <module>
      from pyhive import presto
    File "/root/Python-3.9.15/superset/lib/python3.9/site-packages/pyhive/presto.py", line 22, in <module>
      import requests
  ModuleNotFoundError: No module named 'requests'
  ```

  这次是缺`requests`包。

  解决方法：

  > 按照上面的思路，再次去名为superset的虚拟环境中使用pip install requests，安装requests包。
  >
  > 再重启superset应用之后尝试。

当弹出`Connection looks good!`，表示配置的数据库连接正确了。

![image-20230525135111541](Superset.assets/image-20230525135111541.png)

在`SQL Lab`中执行SQL，验证presto的数据库连接是否真的ok了。

![image-20230525135117654](Superset.assets/image-20230525135117654.png)

