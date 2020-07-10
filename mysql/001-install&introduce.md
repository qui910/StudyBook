# 1 概述

​		数据库技术从传统IT技术开始到现在互联网时代，经历了几个发展阶段。

* 第一阶段，传统IT行业发展阶段，主要就是RDBMS（关系型数据库），其中包括 Oracle，DB2，SQLServer，MySQL等关系型数据库为主。
* 第二阶段，是互联网快速发展阶段，传统关系型数据库已经不能满足业务发展，这时开始兴起NoSQL数据库，其中有Redis，MongoDB，ElaticSearch等。
* 第三阶段，是现在互联网大发展阶段，以前分布在各个子数据中的数据，都合并到一个新型NewSQL数据库中，其中有阿里系的 PalorDB，OB。腾讯的TBSQL，还有pincap的tidb。

网站（http://db-engines.com/en）中有数据库的排名介绍。下面重点讲述：MySQL。

## 1.1 MySQL

​		MySQL主要有几个类型的产品，如Oracle的MySQL，开源的MariaDB，和Perconadb。

​		MySQL现在主流的版本是5.6 和5.7。本次介绍主要使用5.7版本。

​		[官方地址](https://www.mysql.com/)  [中文地址](https://www.mysql.com/cn/)

# 2 安装 

​		MySQL的安装主要有以下几种方式：二进制版本，yum源（ubuntu下可以使用apt-get安装），rpm包，源码包。以下是二进制安装的介绍。[MySQL社区版下载地址](https://dev.mysql.com/downloads/)。[MySQL社区版历史版本地址](https://downloads.mysql.com/archives/community/)。选择：Product Version: 5.7.29   Operating System:Linux - Generic

​		在ubuntu中使用命令下载

```shell
curl -O https://cdn.mysql.com/archives/mysql-5.7/mysql-5.7.29-linux-glibc2.12-x86_64.tar.gz
```

## 2.1 环境准备

```
mkdir -p /data/mysql3307/data
mkdir -p /data/mysql3307/binlog
useradd mysql
```

## 2.2 安装

```shell
#解压软件
sudo tar -zxvf mysql-5.7.29-linux-glibc2.12-x86_64.tar.gz
#建立软连接
sudo ln -s mysql-5.7.29-linux-glibc2.12-x86_64/ mysql
#配置环境变量
sudo vi /etc/profile
	export PATH=/usr/local/mysq/bin:$PATH
source /etc/profile
#初始化库
#创建无密码root
mysqld --initialize-insecure --user=mysql --basedir=/usr/loca/mysql --datadir=/data/mysql3307/data
#创建有密码root，注意这里的密码是临时密码，在控制台显示
mysqld --initialize --user=mysql --basedir=/usr/loca/mysql --datadir=/data/mysql3307/data

# 配置MySQL，MySQL的配置文件一般放置在 /etc目录
cat > /etc/my.cnf <<EOF
[mysqld]
user=mysql
basedir=/usr/loca/mysql
datadir=/data/mysql3307/data
log_bin=/data/mysql3307/binlog/mysql-bin
server_id=7
socket=/tmp/mysql.sock
[mysql]
socket=/tmp/mysql.sock
EOF

#准备启动脚本
cp -a ./support-files/mysql.server  /etc/init.d/mysqld
```

## 2.3 检查

```shell
# 两种方式启动数据
# 第一种
/etc/init.d/mysqld [status|restart|stop|start]

#centos7 
chkconfig --add mysqld
#设置开机启动
chkconfig --level 35 mysqld on   
systemctl [status|restart|stop|start] mysql.service

#登录测试
mysql -uroot -p

#检查端口
netstat -tulnp

#授权密码
mysqladmin -uroot -p passwd 123456
```

ubuntu下设置mysql自启，可以参考：[ubuntu-18.04 设置开机启动脚本](http://www.r9it.com/20180613/ubuntu-18.04-auto-start.html)  同时 chkconfig命令参考:[Linux下chkconfig命令详解](https://www.cnblogs.com/panjun-Donet/archive/2010/08/10/1796873.html)

# 3 MySQL体系结构

![mysql001](.\images\mysql001.jpg)

​		MySQL是典型的C/S架构类型，

​		**客户端：**

* 客户端程序：mysql,mysqladmin,mysqldump...

* API接口方式：C，Java，.Net，Python...

  **连接层：**

* 提供连接协议：网络Socket（TCP/IP）,Unix套接字文件（/tmp/mysql.sock）

* 验证模块：验证用户身份（mysql_native_password）

* 连接线程：接收SQL语句（不处理直接转给SQL层），返回执行结果

```mysql
# 查看连接会话
mysql> show processlist;
+-------+--------+---------------------+-----------+---------+------+----------+------------------+
| Id    | User   | Host                | db        | Command | Time | State    | Info             |
+-------+--------+---------------------+-----------+---------+------+----------+------------------+
| 22291 | root   | localhost           | NULL      | Query   |    0 | starting | show processlist |
| 22292 | wechat | 171.88.178.71:30977 | NULL      | Sleep   |   13 |          | NULL             |
| 22293 | wechat | 171.88.178.71:29760 | wechat_db | Sleep   |   11 |          | NULL             |
+-------+--------+---------------------+-----------+---------+------+----------+------------------+
3 rows in set (0.00 sec)
```

​		SQL层：

* 语法，语义，权限检查
* 语句解析
* 优化基于cost的进行优化
* 执行器 执行SQL

​       存储引擎（plugins storage engine）

## 3.1 专用线程介绍

​		MySQL属于单进程（mysqld），多线程（master thread,IO,SQL,purge）的工作模式。（Oracle就是属于多进程工作模式）

```mysql
# 显示工作线程
mysql> select * from performance_schema.threads;
+-----------+----------------------------------------+------------+----------------+------------------+------------------+--------
--------+---------------------+------------------+-------------------+---------------------------------------------+------------------+------+--------------+---------+-----------------+--------------+| THREAD_ID | NAME                                   | TYPE       | PROCESSLIST_ID | PROCESSLIST_USER | PROCESSLIST_HOST | PROCESS
LIST_DB | PROCESSLIST_COMMAND | PROCESSLIST_TIME | PROCESSLIST_STATE | PROCESSLIST_INFO                            | PARENT_THREAD_ID | ROLE | INSTRUMENTED | HISTORY | CONNECTION_TYPE | THREAD_OS_ID |+-----------+----------------------------------------+------------+----------------+------------------+------------------+--------
--------+---------------------+------------------+-------------------+---------------------------------------------+------------------+------+--------------+---------+-----------------+--------------+|         1 | thread/sql/main                        | BACKGROUND |           NULL | NULL             | NULL             | NULL   
        | NULL                |          7638913 | NULL              | NULL                                        |             NULL | NULL | YES          | YES     | NULL            |         1367 ||         2 | thread/sql/thread_timer_notifier       | BACKGROUND |           NULL | NULL             | NULL             | NULL   
        | NULL                |             NULL | NULL              | NULL                                        |            
....
```

 

# 4 MySQL基础管理

## 4.1 用户管理

### 4.1.1 用户的定义

格式：

```shell
# whitelist（白名单）：能否访问MySQL的地址列表
用户名@'whitelist'    
# 举例
testuser@'localhost' --> 本地能登录用户
testuser@'192.168.10.1' --> 指特定IP等登录用户
testuser@'192.168.10.%' --> 指特定IP地址段等登录用户
testuser@'192.168.10.0/255.255.254.0' --> 指特定IP地址段等登录用户
testuser@'%' -->允许所有地址均可登录
```

### 4.1.2 用户的存储位置

```mysql
mysql> select user,host,authentication_string,plugin from mysql.user;
+------------------+-----------+-------------------------------------------+-----------------------+
| user             | host      | authentication_string（密码）               | plugin（加密方式）      |
+------------------+-----------+-------------------------------------------+-----------------------+
| root             | %         | *02D43CC451497F30127CCCB9A09892CADB5498B8 | mysql_native_password |
...
| debian-sys-maint | localhost | *0F9275073ED7F7E9E0EDCA638955393655200717 | mysql_native_password |
| wechat           | %         | *62141D8A00803D99A862B8C2F01804D95384CB02 | mysql_native_password |
+------------------+-----------+-------------------------------------------+-----------------------+
5 rows in set (0.00 sec)
```

### 4.1.3 用户管理操作

```mysql
# 创建用户，但是密码长度不合规
mysql> create user 'testuser'@'localhost' identified by '123456';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
# 查看 mysql 初始的密码策略
mysql> show variables like 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.00 sec)
# validate_password_length  固定密码的总长度
# validate_password_dictionary_file 指定密码验证的文件路径
# validate_password_mixed_case_count  整个密码中至少要包含大/小写字母的总个数
# validate_password_number_count  整个密码中至少要包含阿拉伯数字的个数
# validate_password_policy 指定密码的强度验证等级，默认为 MEDIUM
## 		关于 validate_password_policy 的取值：
## 		0/LOW：只验证长度
## 		1/MEDIUM：验证长度、数字、大小写、特殊字符
## 		2/STRONG：验证长度、数字、大小写、特殊字符、字典文件
# validate_password_special_char_count 整个密码中至少要包含特殊字符的个数
# set global validate_password_length=6; 修改配置方式
mysql> create user 'testuser'@'localhost' identified by '1234@Abc';
```

```mysql
# 修改用户密码
mysql> alter user testuser@'localhost' identified by '1234@ABc';
Query OK, 0 rows affected (0.00 sec)
# 查看命令帮助
mysql> help alter user;
# 修改用户名
mysql> rename user testuser@'localhost' to test@'localhost';
Query OK, 0 rows affected (0.00 sec)
# 删除用户
mysql> drop user test@'localhost';
Query OK, 0 rows affected (0.00 sec)
```

```mysql
# 查看mysql系统中所有的权限列表
mysql> show privileges;
+-------------------------+---------------------------------------+-------------------------------------------------------+
| Privilege               | Context                               | Comment                                               |
+-------------------------+---------------------------------------+-------------------------------------------------------+
| Alter                   | Tables                                | To alter the table                                    |
| Alter routine           | Functions,Procedures                  | To alter or drop stored functions/procedures          |
...
| Create tablespace       | Server Admin                          | To create/alter/drop tablespaces                      |
| Update                  | Tables                                | To update existing rows                               |
| Usage                   | Server Admin                          | No privileges - allow connect only                    |
+-------------------------+---------------------------------------+-------------------------------------------------------+
31 rows in set (0.00 sec)
# 查询用户的权限列表
mysql> show grants for testuser@'localhost';
+----------------------------------------------+
| Grants for testuser@localhost                |
+----------------------------------------------+
| GRANT USAGE ON *.* TO 'testuser'@'localhost' |
+----------------------------------------------+
1 row in set (0.01 sec)
# 授予wechat_db库所有表的查询权限给用户testuser
mysql> grant select on  wechat_db.* to testuser@'localhost';
Query OK, 0 rows affected (0.00 sec)
mysql> show grants for testuser@'localhost';
+---------------------------------------------------------+
| Grants for testuser@localhost                           |
+---------------------------------------------------------+
| GRANT USAGE ON *.* TO 'testuser'@'localhost'            |
| GRANT SELECT ON `wechat_db`.* TO 'testuser'@'localhost' |
+---------------------------------------------------------+
2 rows in set (0.00 sec)
# 删除权限
mysql> revoke select on wechat_db.* from testuser@'localhost';
Query OK, 0 rows affected (0.00 sec)
mysql> show grants for testuser@'localhost';
+----------------------------------------------+
| Grants for testuser@localhost                |
+----------------------------------------------+
| GRANT USAGE ON *.* TO 'testuser'@'localhost' |
+----------------------------------------------+
1 row in set (0.00 sec)
```



