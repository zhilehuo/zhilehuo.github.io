---
title: ProxySQL 调研记录
date: 2017-04-28 17:31:25
tags:
- SQL
categories:
- Tech
---

> **ProxySQL官网：** http://www.proxysql.com/
> **ProxySQL文档：** https://github.com/sysown/proxysql/wiki

### 背景
阿里云的RDS按连接数售卖，所以，当我们部署了很多的服务实例后，每个实例再配置适当的连接池，就导致了RDS经常连接数过多，因为很多连接是由连接池创建的。而我们在调整多次连接池的参数后，也没有达到满意的效果。

所以，这个情况下的解决方案就是设置一个统一的MySQL代理，所有应用的连接池都连至这个代理，然后代理通过自己的连接池再连接Backend MySQL，这样能够更有效的利用连接资源。

> 几个可选的项目概况：
> **MySQL Proxy**
> MySQL官方品牌，但据说很久不维护了，以及稳定性一直被诟病
> **Atlas**
> 360在MySQL Proxy基础上修改的一个版本，据说性能比MySQL Proxy强大不少，目前已经不维护了
> **ProxySQL**
> 国外的资料中使用比较多的一个项目
> **MaxScale**
> 同样是国外使用较多的一个项目，经常拿来和ProxySQL做对比，开源版本最多支持2个Backend DB
>
> 最后综合了各种资料后，还是决定试用ProxySQL。

### 安装
官网下载RPM安装包后，直接安装。

我在安装时，被提示缺少`perl-MySQL(DBD)`组件，直接用`yum`安装即可：
```bash
[root@server3 download]# rpm -ivh proxysql-1.3.6-1-centos67.x86_64.rpm 
error: Failed dependencies:
	perl(DBD::mysql) is needed by proxysql-1.3.6-1.x86_64
[root@server3 download]#
[root@server3 download]# yum install perl-DBD-MySQL
...
```

安装完成后，ProxySQL自动创建了`/etc/init.d/proxysql`脚本用来控制服务启动，同时也将ProxySQL加到了`service`中，可以通过service命令控制。

默认安装后的配置文件为`/etc/proxysql.cnf`

### 使用前奏
使用之前要说一下ProxySQL的配置方式。

ProxySQL的配置分为3层：`Runtime`/`Memory`/`Disk`，这3层可以理解为ProxySQL配置的3个复本。其中，Runtime是正在生效的配置；Memory是内存中的配置，内存中的配置被修改后，不会反应在运行状态中，而是需要手动加载，才可以覆盖运行中的配置（即Runtime配置）；Disk则是持久化的配置，Memory中的配置只在手动刷新到Disk中，才可以被保存，否则服务重启后配置就会丢失。

ProxySQL所有的配置都支持在运行时修改(除了两个配置，一个是线程数，还有一个是什么来着，忘了~)，详细可以参考 [Multi layer configuration system][1]。

Disk配置分为配置文件和db文件两种，db文件位于ProxySQL的datadir中。ProxySQL在启动时，优先加载datadir中的db文件，如果db文件不存在，就加载配置文件。

### 首次启动
可以使用`service proxysql start`来启动ProxySQL。默认加`载/etc/proxysql.cnf`，这个配置文件中默认指定`/var/lib/proxysql`目录为datadir，这个datadir中已经默认存在了`proxysql.db`文件，所以，ProxySQL此时会自动加载db文件。

服务启动后，看一下进程信息：
![67cf0be85be34ef89ebd60beb0461bbe-1493368198897.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/67cf0be85be34ef89ebd60beb0461bbe-1493368198897.png) 

ProxySQL启动后，默认开启两个端口：`6032`和`6033`。其中，6032是管理端口，6033是数据端口。现在什么都没配，所以数据端口没有合法用户，是连接不上的，因此可以先来连接一下管理端口：

    mysql -uadmin -padmin -h127.0.0.1 -P6032

默认用户名和密码都是`admin`。连接成功后可以`show databases`看一下：
```bash
[root@server3 ~]# mysql -uadmin -padmin -h127.0.0.1 -P6032
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 17
Server version: 5.5.30 (ProxySQL Admin Module)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+-----+---------+-------------------------------+
| seq | name    | file                          |
+-----+---------+-------------------------------+
| 0   | main    |                               |
| 2   | disk    | /var/lib/proxysql/proxysql.db |
| 3   | stats   |                               |
| 4   | monitor |                               |
+-----+---------+-------------------------------+
4 rows in set (0.01 sec)

mysql> 
```

其中 **main** 库用来配置各种参数。可以看看当前的配置：
```
mysql> select * from main.global_variables;
+---------------------------------+----------------+
| variable_name                   | variable_value |
+---------------------------------+----------------+
| mysql-shun_on_failures          | 5              |
| mysql-shun_recovery_time_sec    | 10             |
| mysql-query_retries_on_failure  | 1              |
| mysql-connect_retries_delay     | 1             
......
```

### 如何修改配置
先看看 **main** 库中都有哪些表：
```
mysql> show tables;
+--------------------------------------+
| tables                               |
+--------------------------------------+
| global_variables                     |
| mysql_collations                     |
| mysql_query_rules                    |
| mysql_replication_hostgroups         |
| mysql_servers                        |
| mysql_users                          |
| runtime_global_variables             |
| runtime_mysql_query_rules            |
| runtime_mysql_replication_hostgroups |
| runtime_mysql_servers                |
| runtime_mysql_users                  |
| runtime_scheduler                    |
| scheduler                            |
+--------------------------------------+
13 rows in set (0.00 sec)

mysql> 
```

`runtime_`打头的都是运行是配置，这是不可以修改的；`global_variables`和`mysql_`打头的表，是Memory配置表。
修改Memory其实就是修改这几张表中的数据，其中：
- `global_variables`表，是运行的基础配置
- `mysql_users`表，是授权用户配置
- `mysql_servers`表，是Backend MySQL配置
- `mysql\_qurey\_rules`表，是查询策略配置（这个我们暂时使用不到）

修改`global_variables`表时，也支持使用`SET`语法。

根据之前所述，Memory配置修改后，是不会在线上生效的，需要用如下指令使其生效：

    mysql>LOAD MYSQL SERVERS TO RUNTIME;

再看看如何将Memory刷到DISK：

    mysql>SAVE MYSQL SERVERS TO DISK;

这两步千万不要忘，尤其是刷Disk，很容易被遗忘。

上述命令中的`mysql servers`，还有几类可选值，分别为：
- mysql users
- mysql variables
- admin variables
- mysql query rules

#### 添加Backend MySQL
将Backend MySQL添加至`mysql_servers`表：
```
mysql> INSERT INTO mysql_servers (hostname) VALUES ('10.168.21.138');
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM mysql_servers\G
*************************** 1. row ***************************
       hostgroup_id: 0
           hostname: 172.16.0.1
               port: 3306
             status: ONLINE
             weight: 1
        compression: 0
    max_connections: 1000
max_replication_lag: 0
            use_ssl: 0
     max_latency_ms: 0
            comment:
1 row in set (0.00 sec)
```

未设置的字段都以默认值填充。其中，`hostgroup_id`相当于后端分组，在我们的场景中也暂时用不到。

#### 添加User
这里添加的用户，就是应用程序连接ProxySQL时使用的用户，同时也是ProxySQL连接后端的用户信息。用户的密码默认以明文存储，如果想以摘要存储，可以参考[Passwords management](https://github.com/sysown/proxysql/wiki/Passwords-management)

```
mysql> INSERT INTO mysql_users(username,password) VALUES ('nianxy','nianxy');
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM mysql_users \G;
*************************** 1. row ***************************
              username: nianxy
              password: nianxy
                active: 1
               use_ssl: 0
     default_hostgroup: 1
        default_schema: NULL
         schema_locked: 0
transaction_persistent: 0
          fast_forward: 0
               backend: 1
              frontend: 1
       max_connections: 10000
1 row in set (0.00 sec)

ERROR: 
No query specified
```

#### 设置Global Variables
变量较多，挑一堆有用的列举一下，其它可以参考[Global variables](https://github.com/sysown/proxysql/wiki/Global-variables)：
**admin-admin_credentials**
管理端口用户名和密码
**admin-mysql_ifaces**
管理端口
**admin-stats_credentials**
数据端口用户名和密码
**mysql-commands_stats**
是否开启SQL统计。开启后会分析每条SQL语句。默认开启。
**mysql-connection_max_age_ms**
到Backend的连接空闲多久后会自动关闭。默认1秒。
**mysql-default_query_timeout**
到Backend的查询超时时间，超过后会主动停止查询，并从Backend Kill掉该连接。默认24小时。
**mysql-free_connections_pct**
允许的Backend空闲连接数，是一个占mysql-max_connections数量的百分比，默认是10。
**mysql-interfaces**
数据端口配置
**mysql-max_connections**
ProxySQL可接收的最大连接数。默认10000。
**mysql-server_version**
ProxySQL返回给客户端的MySQL版本号，有可能影响客户端行为。默认5.5.30。
**mysql-session_idle_show_processlist**
管理端口进行show processlist时，是否显示空闲连接，开启后会影响性能。默认关闭。
**mysql-wait_timeout**
客户端连接空闲超时时间，默认8小时。

以上请注意`mysql-connection_max_age_ms`和`mysql-wait_timeout`的区别，前者是到Backend的超时，后者是到客户端的超时。

### 启动多个实例
ProxySQL中配置的所有Backend会认为在数据上是等效的，即数据都是同步的，所以，当我们需要连接多个不同的MySQL服务时，就需要启动多个ProxySQL实例了。

根据前边的内容，其实，只要指定不同的Datadir或Conf，就可以实现多实例启动了。ProxySQL在安装时，自动安装了`/etc/init.d/proxysql`脚本，通过观察这个脚本，发现只要修改脚本头部的`DATADIR`变量的值，即可指定ProxySQL启动时的Datadir。

当我们首次指定一个Datadir时，因为目录为空，所以其中不会包含db配置文件，此时，ProxySQL就会根据`/etc/proxysql.cnf`来初始化ProxySQL实例，并在目录中生成db文件，以后就可以通过修改db文件，来完成ProxySQL的配置了。需要注意的是，下次再启动实例时，`/etc/proxysql.cnf`也失去作用了。

综上，理论上我们通过拷贝`/etc/init.d/proxysql`，并修改里边的`DATADIR`，即可实现多实例启动了。

不过，实际操作时发现`/etc/init.d/proxysql`对多实例支持并不友好，在停止服务时，它会将所有proxysql进程全部杀死，所以我们需要改造一下这个脚本，具体如下：
1. 将头部`DATADIR`变量，修改为实际使用的目录
2. 将头部`OPTS`变量中的`-c /etc/proxysql.cnf`去掉，我们用不到配置文件
  ![9b4b2e11b0724368a9c54f46c59d3aa0-1493374029201.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/9b4b2e11b0724368a9c54f46c59d3aa0-1493374029201.png) 
3. 在第32行左右，getpid这个函数里，找到`pid=ps -p $pid | grep ...`这一行，修改ps命令的参数为`pid=ps -o pid,ppid --ppid $pid | grep ...`
  ![0ccfa0e2c22b4fcd9113abc6d98a8a0d-1493374170805.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/0ccfa0e2c22b4fcd9113abc6d98a8a0d-1493374170805.png) 
4. 在第97行左右，stop这个函数里，将`for i in pgrep proxysql...`这个循环整体注释掉，替换为`kill $pid`
  ![368423d0114c441484a5131fbd4c6616-1493374201255.png](http://onpyrjcca.bkt.clouddn.com//file/2017/4/368423d0114c441484a5131fbd4c6616-1493374201255.png) 


[1]: https://github.com/sysown/proxysql/wiki/Multi-layer-configuration-system