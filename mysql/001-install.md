# 1 概述

​		MySQL是最流行的开源SQL数据库管理系统，它由MySQL AB开发、发布和支持。MySQL AB是一家由MySQL开发人员创建的商业公司，它是一家使用了一种成功的商业模式来结合开源价值和方法论的第二代开源公司。MySQL是MySQL AB的注册商标。

​		MySQL是一个快速的、多线程、多用户和健壮的SQL数据库服务器。MySQL服务器支持关键任务、重负载生产系统的使用，也可以将它嵌入到一个大配置(mass-deployed)的软件中去。

​		MySQL网站(http://www.mysql.com)提供了关于MySQL和MySQL AB的最新的消息。

[官方文档8.0](https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html)

​		MySQL具有如下特点或特性：

* MySQL是一个数据库管理系统；
* MySQL是一个关系数据库管理系统；
* MySQL是开源的；
* MySQL服务器是一个快的、可靠的和易于使用的数据库服务器；
* MySQL服务器工作在客户/服务器或嵌入系统中；

​		MySQL有两种安装方式：**源码包安装**和**二进制包安装**。这两种方式各有特色：二位制包安装不需编译，针对不同的平台有经过优化编译的不同的二进制文件以及包格式，安装简单方便；源码包则必须先配置编译再安装，可以根据你所用的主机环境进行优化，选择最佳的配置值，安装定制更灵活。下面分别介绍这两种安装方式。

​		Linux系统下，mysql的配置参数文件为my.cnf，一般按下面的顺序查找此文件：/etc目录、mysql安装目录、mysql数据目录。配置模板位于源码树的support-files目录，有my-small.cnf、my-medium.cnf、my-large.cnf、my-huge.cnf四个

# 2 二进制安装

​		以MySQL8.0为例，说明安装

## 2.1 清除旧版本

​		检查MySQL及相关RPM包，是否安装，如果有安装，则移除。以Centos为例

```shell
rpm -qa | grep -i mysql
yum -y remove 包名 或者 rpm -e 包名 --nodeps (nodeps为不考虑依赖，根据情况确定是否添加)
rpm -e mysql57-community-release-el7-9.noarch 
rpm -e mysql-community-server-5.7.17-1.el7.x86_64 
rpm -e mysql-community-libs-5.7.17-1.el7.x86_64 
rpm -e mysql-community-libs-compat-5.7.17-1.el7.x86_64 
rpm -e mysql-community-common-5.7.17-1.el7.x86_64 
rpm -e mysql-community-client-5.7.17-1.el7.x86_64 
```

​		删除老版本mysql的开发头文件和库

```shell
rm -fr /usr/lib/mysql  
rm -fr /usr/include/mysql
```

​		卸载后/var/lib/mysql中的数据及/etc/my.cnf不会删除，如果确定没用后就手工删除

```shell
rm -f /etc/my.cnf
rm -fr /var/lib/mysql
```

​		清除剩余

```shell
whereis mysql 
mysql: /usr/bin/mysql /usr/lib64/mysql /usr/local/mysql /usr/share/mysql /usr/share/man/man1/mysql.1.gz 
# 删除上面的文件夹 
```

​		删除配置

```shell
rm -rf /usr/my.cnf 
rm -rf /root/.mysql_sercret 
```

​		剩余配置检查

```shell
chkconfig --list | grep -i mysql 
chkconfig --del mysqld 
# 根据上面的列表,删除 ,如:mysqld
```





