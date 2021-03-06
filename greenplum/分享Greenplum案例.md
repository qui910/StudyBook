# 1 概述		

​		大家好！我是来自个推的栗子，感谢Greenplum中文社区！今天我给大家讲一讲个推在Greenplum中的实践。那我们就直接进入今天的分享主题《个推如何采用Greenplum提高PB级别数据处理能力》

​		近年来，移动互联网、物联网、云计算的快速发展，催生了海量的数据。在大数据处理方面，不同技术栈所具备的性能也有所不同。如何快速有效地处理这些体量庞大的数据，令不少开发者为之苦恼。随着Greenplum的异军突起，以往大数据仓库所存在的很多问题都得到了有效解决，Greenplum也成为新一代数据库的典型代表。

​		今天，我将就个推在处理庞大的数据量时，如何选择有效的技术栈进行介绍，并结合自身业务场景，分析一下Greenplum在个推中的实践。

​		首先，我们来看一下Greenplum诞生的背景。

​		2002年，互联网数据量正处于快速增长期，一方面传统数据库难以满足当前的计算需求，另一方面传统数据库大多基于SMP架构，扩展性能差。因此面对日益增长的数据量，SMP架构难以继续支撑，开发者需要一种数据库，可以支持分布式并行数据计算能力，Greenplum便应运而生。

​		大家都知道，和传统数据库的SMP架构不同，Greenplum是一种完全无共享(Share Nothing)的结构，相比SMP，扩展能力明显提升。大家可以看一下这张图。Greenplum系统主要基于MPP架构，由多个服务器通过节点互联网络连接而成，每个节点只需访问自己的本地资源，包括内存、存储等。

![20200613142330](images\20200613142330.png)

​		在Greenplum中，Master上有主节点和从节点两部分，两者主要的功能是生成查询计划、派发、协调Segment并行计算，同时通过Master维护global system catalog。 global system catalog这个全局目录存着一组Greenplum数据库系统本身所具有的元数据的系统表。Master通常不参与数据交互，Greenplum所有的并行任务都是在Segment的数据节点上完成的。因此，Master节点不会成为数据库的性能瓶颈。

​		中间的网络层Interconnect，主要负责并行查询计划Dispatch分发，同时通过libpq网络连接协调QE节点上     执行器的并行执行。正是因为Interconnect的存在，Greenplum才能实现对同一个集群中多个PostgreSQL实例的高效协同和并行计算。

​		结构图下方是负责数据存储和计算的节点，每个节点上多个实例。每个实例都是一个PostgreSQL数据库，同一机器上的实例共享节点的IO和CPU，而不同机器间相互独立。PostgreSQL在稳定性和数据处理性能方面较优，同时又有丰富的语法支持，满足了Greenplum的功能需要。

​		关于Greenplum的架构，如果大家想了解更详细的内容，可以去看一下之前社区做的内核分享，Greenplum中文社区的B站频道上是有视频可以看回放的。

​		接着，让我们再来看一下Greenplum的优势在哪里。

# 2 优势

​		优势一：支持数据快速加载和并行计算，大幅提升数据处理效率

![20200613142744](images\20200613142744.png)		Greenplum的数据管道可以高效地将数据从磁盘传输到CPU，而目前市面上常用的计算引擎Spark在传输数据时，则需要为每个并发查询分配一个内存，这对大型数据集的查询十分不利。Greenplum能够并行加载数据，具备实时查询功能，可以对大数据集进行更高效的计算。

​		优势二：扩展性能增强

​		Greenplum基于MPP架构，节点之间完全不共享，同时又可以并行查询，因此其在进行线性扩展时，数据规模可以达到PB级别。目前，Greenplum已经实现了开源，并且社区生态活跃，对于使用者而言，也是极为可靠的。

​		优势三：功能性优化

​		Greenplum支持复杂的SQL查询，大幅简化了数据的操作和交互过程，而目前流行的HAWQ、Spark SQL、Impala等技术基本都是基于MapReduce所进行的优化，虽然部分技术也使用了SQL查询，但是对SQL的支持比较有限。

​		我们针对目前市面上的几款主流工具，进行了多维度的对比。大家可以看一下下面这张图，在图中，我们分别对Greenplum, phoenix，Impala+kudu的使用方式，优点和缺点进行了对比。Greenplum确实是一款非常优秀的数据库产品。

![20200613143003](images\20200613143003.png)

# 3 容错机制

​		现在，让我们再来看一下Greenplum的容错机制

​		Greenplum数据库简称GPDB，它拥有丰富的特性，支持多级容错机制，具备高可用性能。 下面我们将从三点来介绍下Greenplum的容错机制。

​	1. 主节点高可用：为了避免主节点单点故障问题，Greenplum特别设置了主节点的副本（称为Standby Master），通过流复制技术实现两者同步复制。当主节点发生故障时，从节点可以成为主节点，完成用户请求并协调查询执行。

  2. 数据节点高可用：Greenplum每个数据节点都可以配备一个镜像，在6版本之前其通过文件块级别的同步来实现数据同步；在6版本时，Greenplum采用WAL日志复制的方式来实现数据同步（称为filerep技术）。故障检测进程（ftsprobe）会定期探测各个数据节点的心跳，当某个节点发生故障时，GPDB会自动进行故障切换。

  3. 网络高可用：为了避免网络的单点故障，每个主机会配置多个网口，并使用多个交换机，防止网络故障时，整个服务器都不可用。

​		同时，Greenplum具有图形化的性能监控功能。基于此功能，用户可以确定数据库当前的运行情况和历史查询信息，同时跟踪系统使用情况和资源信息。

# 4 应用场景

​	介绍完Greenplum的诞生背景，架构，优势和容错机制，现在我们来讲解一下Greenplum在个推业务场景中的应用。![20200613144514](images\20200613144514.png)

​		在深入调研了Greenplum之后，我们将它纳入了个推的技术地图中，也在个推的业务线中进行了实践，比如在 “个推应用统计”的业务中。现在，让我们分别从业务痛点、解决方案、和使用情况三大块来具体介绍个推是如何应用Greenplum的。

## 4.1 业务痛点

​		“个推应用统计”是一款移动APP数据统计分析平台，它能从用户属性、使用行为、行业对比等多指标多维度对APP进行全面统计分析，帮助APP运营者深层次挖掘用户需求，清晰地了解APP所处的行业地位，为产品运营和推广决策提供全方位数据支撑。		

​		最开始，“个推应用统计”中的数据统计主要是用离线统计的方式，后来随着产品的逐渐迭代、优化，很多需求逐渐无法满足了，例如：

1. 部分场景需要实时计算；
2. 用户活跃统计中跨天去重的场景；
3. 指标的统计维度过多，无法提前计算。![20200613144659](images\20200613144659.png)

## 4.2 解决方案

​		针对这一系列问题，个推引入了Greenplum作为实时数据分析的工具。目前个推使用的是Greenplum 5.16的版本。

​		首先，针对实时性的问题。“个推应用统计”每天的事件日志量达几十亿条，在数据导入的时候，我们使用了gpkafka作为实时数据写入的方式。

​		数据导入的具体流程：

1. Flume采集日志数据后发送到Kafka集群；
2. Spark Streaming先消费Kafka集群中的数据，再进行实时数据清洗、处理；
3. Spark Streaming随后将处理后的数据写入到Kafka集群中；



​		gpkafka工具消费Kafka集群中的数据，把消费后的数据写入到Greenplum的Heap表中，进行实时的数据分析展示。

![20200613144930](images\20200613144930.png)

​		接着，针对跨天去重的问题。我们改造了Greenplum，在其基础上融入了Roaringbitmap。在数据处理阶段，定时任务将每天的用户信息构造成bitmap的格式，根据bitmap的“与、或”运算实现多天的用户计算查询。

​		针对复杂维度统计的场景。“个推应用统计”有一些场景，用户会定义复杂的查询条件，比如多维事件分析、漏斗查询等。 “个推应用统计” 查询功能模块，运用了pljava，类似于一个udf脚本，通过在SQL中使用pljava定义的function，实现了复杂统计的快速查询。

# 5 使用情况

​		最后，我们来介绍一下个推的使用情况。

​		目前个推业务线使用的Greenplum集群规模是10台，每天的数据量达几十亿。在近期，我们也经历了集群的扩容，通过Greenplum中的原生tools对节点进行扩充，对数据进行了自动重分布。Greenplum的一系列自动化工具也让运维人员的的扩容工作变得更加便捷。

​		我们来说说个推引用Greenplum后，效果如何。

![20200613145200](images\20200613145200.png)

​		引入Greenplum，极大地增强了个推的数据处理能力，包括海量数据的实时入库、实时查询、标签运算等。当前，数据的高效使用对于企业的重要性不言而喻。构建OLAP分析系统，对于业务的指导、公司的决策，都有举足轻重的作用，而Greenplum正可以帮助企业快速构建分析引擎，实现全面的数据分析。大家可以看一下下图中的使用前后的对比。

![20200613145243](images\20200613145243.png)

​		这就是我们今天分享的全部内容， 我们首先介绍了Greenplum的诞生背景，解读了Greenplum的架构，分析了Greenplum的优势。接着我们从业务痛点、解决方案、使用情况、和效果四个方面介绍了个推在Greenplum方面的实践。

​		未来，个推也将对Greenplum进行更深入地研究，例如自定义函数的使用方式，与开发者一同分享如何在生产环境中更好地对Greenplum进行使用。

​		谢谢大家！今天的分享结束了！期待Greenplum中文社区的更多分享内容！大家如果有什么问题也可以在群里留言。



