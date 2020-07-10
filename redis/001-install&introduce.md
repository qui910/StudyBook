# 1 概述

​		首先我们要知道，磁盘中的一个扇区为512字节（byte=8bit），也就是磁盘读取的最小单位。而现代的磁盘容量不断变大，512字节的设计不再合理，这时就出现了4K对齐的概念，即可以简单理解为扇区变为了4096字节（4096/1024=4K）。（具体关于4K对齐的文章可以参考[什么叫4K对齐、如何进行硬盘4K对齐？](https://blog.csdn.net/hyz301/article/details/64130411/)）。

​		下面讲下以Oracle数据库说明下关系型数据库存储磁盘数据时的关系，首先在Oracle初始建库时，要设置`DB_BLOCK_SIZE`，其可以设置为 4K、8K、16K、32K、64K等几种大小（ORACLE的物理文件最大只允许4194304个数据块（由操作系统决定），表空间数据文件的最大值为 `4194304×DB_BLOCK_SIZE/1024M`）。这里的`DB_BLOCK_SIZE`就是数据库操作的最小单位。如此设计的目的就是保证数据库块为4K时，正好可以写满一个磁盘的扇区（4K）。从而在检索磁盘文件时，减少磁盘寻址的时间。

​		在磁盘文档不断的增大后，虽然使用4K的寻址方式，但是检索速度还是会不断下降，这是就出现了数据库索引`index`的概念。`index`的最小存储单位也是4K，但是这个4K的datapage中存储的就是多个数据库datapage的真实检索信息。这样检索数据快时，可以先通过`index`快速定位到较小的范围，而不必全部检索。同时，多个`index`块的相关检索地址还会存储到内存中，即B+Tree索引（内存中只存储`index`块的检索信息（树枝），具体`index`块（叶子节点）还是存储在磁盘中），从而更快的加速检索速度。

![redis-001](.\images\redis-001.png)

​		在数据量逐渐增大后，会造成插入，更新，删除数据变慢，因为在这3个过程中是需要维护索引的。而针对单个数据块的检索是不会变慢的，同时多并发查询或复杂SQL查询时是会变慢的。

​		鉴于大数据量后，传统关系型数据库会逐渐的变慢，查询效率遍地。由此引发出的，就是考虑使用内存数据库（[SAP HANA](https://db-engines.com/en/system/SAP+HANA)），然后纯内存数据库的价格非常昂贵。所以在互联网高速发展的今天，人们就采取了一种折中方案，即缓存数据库（`redis`，`memcached`）。

# 2 Redis

​		在进行缓存数据库的选型时，可以先登录下述网址看下数据库的整体排名和各自的特点（[db-engines](https://db-engines.com/en/)），在做技术选型时，一定要注意各种数据库的特性。在上述网址（点击[DB-Engines Ranking](https://db-engines.com/en/ranking)），详细点击要选型的数据库有各自的详细介绍。

​		Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 [字符串（strings）](http://redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://redis.cn/topics/lru-cache.html)，[事务（transactions）](http://redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。

![redis-002](.\images\redis-002.png)		

​		而同为key-value数据库的memcached，value是无类型区分的。虽然可以使用JSON来保存各种类型数据，但是使用Redis的话，它本身会内置很多类处理的函数，这样减少了客户端查询某种类型数据时候的操作和流量，比处理JSON快捷和迅速。

## 2.1 下载安装

​		下载Redis，首先访问[官网](https://redis.io/)，或[中文官网](http://www.redis.cn/)。这里下载后的都为源码包，需要在Linux中通过编译安装。

