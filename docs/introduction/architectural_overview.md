
JanusGraph是一个图形数据库引擎。 JanusGraph本身专注于压缩图序列化、丰富图数据建模、高效的查询执行。 此外，JanusGraph利用Hadoop进行图分析和批处理。JanusGraph为数据持久化，数据索引和客户端访问实现了强大的模块化接口。

JanusGraph的模块化架构使其能够与各种存储，索引和客户端技术进行互操作; 这也使得JanusGraph升级对应的组件过程变得更加简单。

在JanusGraph和磁盘之间有一个或多个存储和索引适配器。 JanusGraph标配以下适配器，但JanusGraph的模块化架构支持第三方适配器。

数据存储:

* Apache Cassandra
* Apache HBase
* Oracle Berkeley DB Java企业版

索引，用于加快访问速度并支持更复杂的查询语句:

* Elasticsearch
* Apache Solr
* Apache Lucene

总体来讲，应用程序可以通过两种方式与JanusGraph进行交互：

* 嵌在应用程序中的JanusGraph在同一个JVM中执行Gremlin语句。 查询任务、JanusGraph缓存和事务处理都在同一个JVM中，而后端数据检索可能是在本地或远程。
* 通过向服务器提交Gremlin查询语句来与本地或远程JanusGraph实例交互。 JanusGraph本身支持Apache TinkerPop栈的Gremlin Server组件。

图 2.1.JanusGraph系统架构图
![](../img/architecture-layer-diagram.svg)