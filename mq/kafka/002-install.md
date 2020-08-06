# 1 安装环境

## 1.1 JDK安装

略

```shell
export JAVA_HOME=/usr/local/java
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=.:${JAVA_HOME}/bin:$PATH
```

## 1.2 SSH免密码登录

```shell
# 生成当前节点秘钥和公钥
ssh-keygen -t rsa

# 将公钥id_rsa.pub文件中内容追加到authorized_keys文件中
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# 文件赋权
chmod 600 ~/.ssh/authorized_keys
```



## 1.3 安装与配置Zookeeper

```shell
# 配置zookeeper配置文件
cp zoo_sample.cfg zoo.cfg

# 在dataDir目录下创建myid文件，并写入数字1 对应cfg文件中的server.1

# 修改配置文件
export ZK_HOME=/data/opt/zookeeper
export PATH=$PATH:$ZK_HOME/bin

# 启动脚本
zkServer.sh start
```

# 1.4 部署Kafka

### 1.4.1 单机模式

```shell
# 配置环境变量
export KAFKA_HOME=/data/opt/kafka
export PATH=$PATH:$KAFKA_HOME/bin

# 配置kafka中server.properties文件

# 启动
kafka-server-start.sh $KAFKA_HOME/config/server.properties &
```

### 1.4.2 分布式模式



### 1.4.3 安装Kafka Eagle监控



# 2 控制器

控制器，其实就是Kafka系统的一个代理节点，它具有一般代理节点的功能，同时具有选举主题分区Leader节点的功能。在Kafka启动时，其中的一个代理节点会被选为控制器，负责管理主题分区和副本状态，还会执行分区重新分配的管理任务。

## 2.1 控制器启动顺序

