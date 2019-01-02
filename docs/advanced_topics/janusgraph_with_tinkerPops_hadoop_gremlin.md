第37章 JanusGraph与TinkerPop的Hadoop-Gremlin
---
本章介绍如何利用Apache Hadoop和Apache Spark配置JanusGraph以进行分布式图形处理。这些步骤将概述如何开始这些项目，但请参考这些项目社区以更熟悉它们。

JanusGraph-Hadoop与TinkerPop的hadoop-gremlin软件包一起用于通用OLAP。

对于下面示例的范围，Apache Spark是计算框架，Apache Cassandra是存储后端。可以使用其他包来跟踪指示，只需对配置属性进行微小更改。
>注意:
本章中的示例基于在本地模式或独立群集模式下运行Spark。在YARN或Mesos上使用Spark时需要进行其他配置。

### 37.1 配置Hadoop以运行OLAP
要从Gremlin控制台运行OLAP查询，需要满足几个先决条件。您需要将Hadoop配置目录添加到类路径中，配置目录需要指向一个活动的Hadoop集群。

Hadoop提供分布式访问控制的文件系统。运行在不同计算机上的Spark工作程序使用Hadoop文件系统来获得基于文件的操作的公共源。各种OLAP查询的中间计算可以保留在Hadoop文件系统上。

有关配置单节点Hadoop集群的信息，请参阅官方Apache Hadoop文档。

一旦启动并运行了Hadoop集群，我们就需要在CLASSPATH中指定Hadoop配置文件。下面的文档希望您的配置文件位于/etc/hadoop/conf下。

验证后，按照以下步骤将Hadoop配置添加到CLASSPATH并启动Gremlin控制台，它将扮演Spark驱动程序的角色。
```
export HADOOP_CONF_DIR=/etc/hadoop/conf
export CLASSPATH=$HADOOP_CONF_DIR
bin/gremlin.sh
```
将Hadoop配置路径添加到类路径后，我们可以通过以下快速步骤验证Gremlin控制台是否可以访问Hadoop集群:
```
gremlin> hdfs
==>storage[org.apache.hadoop.fs.LocalFileSystem@65bb9029] // BAD

gremlin> hdfs
==>storage[DFS[DFSClient[clientName=DFSClient_NONMAPREDUCE_1229457199_1, ugi=user (auth:SIMPLE)]]] // GOOD
```
### 37.2 OLAP遍历
JanusGraph-Hadoop使用TinkerPop的hadoop-gremlin包进行通用OLAP遍历图，并通过利用Apache Spark并行化查询。

#### 37.2.1 使用Spark Local进行OLAP遍历
这里演示的后端是Cassandra，下面是OLAP示例。将需要特定于该存储后端的附加配置。配置由gremlin.hadoop.graphReader属性指定，该属性指定从存储后端读取数据的类。

JanusGraph目前支持以下graphReader类：
- Cassandra3InputFormat 与Cassandra 3一起使用
- CassandraInputFormat 与Cassandra 2一起使用
- HBaseInputFormat和HBaseSnapshotInputFormatHBase一起使用
以下属性文件可用于连接Cassandra中的JanusGraph实例，以便它可以与HadoopGraph一起使用来运行OLAP查询。

```
# read-cassandra-3.properties
#
# Hadoop Graph Configuration
#
gremlin.graph=org.apache.tinkerpop.gremlin.hadoop.structure.HadoopGraph
gremlin.hadoop.graphReader=org.janusgraph.hadoop.formats.cassandra.Cassandra3InputFormat
gremlin.hadoop.graphWriter=org.apache.tinkerpop.gremlin.hadoop.structure.io.gryo.GryoOutputFormat

gremlin.hadoop.jarsInDistributedCache=true
gremlin.hadoop.inputLocation=none
gremlin.hadoop.outputLocation=output
gremlin.spark.persistContext=true

#
# JanusGraph Cassandra InputFormat configuration
#
# These properties defines the connection properties which were used while write data to JanusGraph.
janusgraphmr.ioformat.conf.storage.backend=cassandra
# This specifies the hostname & port for Cassandra data store.
janusgraphmr.ioformat.conf.storage.hostname=127.0.0.1
janusgraphmr.ioformat.conf.storage.port=9160
# This specifies the keyspace where data is stored.
janusgraphmr.ioformat.conf.storage.cassandra.keyspace=janusgraph
# This defines the indexing backned configuration used while writing data to JanusGraph.
janusgraphmr.ioformat.conf.index.search.backend=elasticsearch
janusgraphmr.ioformat.conf.index.search.hostname=127.0.0.1
# Use the appropriate properties for the backend when using a different storage backend (HBase) or indexing backend (Solr).

#
# Apache Cassandra InputFormat configuration
#
cassandra.input.partitioner.class=org.apache.cassandra.dht.Murmur3Partitioner

#
# SparkGraphComputer Configuration
#
spark.master=local[*]
spark.executor.memory=1g
spark.serializer=org.apache.spark.serializer.KryoSerializer
spark.kryo.registrator=org.apache.tinkerpop.gremlin.spark.structure.io.gryo.GryoRegistrator
```
首先使用上述配置创建属性文件，然后在Gremlin控制台上加载它以运行OLAP查询，如下所示：
```
bin/gremlin.sh

         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----
plugin activated: janusgraph.imports
gremlin> :plugin use tinkerpop.hadoop
==>tinkerpop.hadoop activated
gremlin> :plugin use tinkerpop.spark
==>tinkerpop.spark activated
gremlin> // 1. Open a the graph for OLAP processing reading in from Cassandra 3
gremlin> graph = GraphFactory.open('conf/hadoop-graph/read-cassandra-3.properties')
==>hadoopgraph[cassandra3inputformat->gryooutputformat]
gremlin> // 2. Configure the traversal to run with Spark
gremlin> g = graph.traversal().withComputer(SparkGraphComputer)
==>graphtraversalsource[hadoopgraph[cassandra3inputformat->gryooutputformat], sparkgraphcomputer]
gremlin> // 3. Run some OLAP traversals
gremlin> g.V().count()
......
==>808
gremlin> g.E().count()
......
==> 8046
```

### 37.2.2 使用Spark Standalone Cluster进行OLAP遍历
上一节中介绍的步骤也可以与Spark独立群集一起使用，只需进行少量更改：

- 更新spark.master属性以指向Spark主URL而不是本地URL
- 更新spark.executor.extraClassPath以启用Spark执行程序以查找JanusGraph依赖项jar
- 将JanusGraph依赖项jar复制到每个Spark执行器计算机上一步中指定的位置

>注意:
我们将janusgraph-distribution/lib下的所有jar复制到/opt/lib/janusgraph /中，并在所有worker中创建相同的目录结构，并在所有worker中手动复制jar。

用于OLAP遍历的最终属性文件如下：
```
# read-cassandra-3.properties
#
# Hadoop Graph Configuration
#
gremlin.graph=org.apache.tinkerpop.gremlin.hadoop.structure.HadoopGraph
gremlin.hadoop.graphReader=org.janusgraph.hadoop.formats.cassandra.Cassandra3InputFormat
gremlin.hadoop.graphWriter=org.apache.tinkerpop.gremlin.hadoop.structure.io.gryo.GryoOutputFormat

gremlin.hadoop.jarsInDistributedCache=true
gremlin.hadoop.inputLocation=none
gremlin.hadoop.outputLocation=output
gremlin.spark.persistContext=true

#
# JanusGraph Cassandra InputFormat configuration
#
# These properties defines the connection properties which were used while write data to JanusGraph.
janusgraphmr.ioformat.conf.storage.backend=cassandra
# This specifies the hostname & port for Cassandra data store.
janusgraphmr.ioformat.conf.storage.hostname=127.0.0.1
janusgraphmr.ioformat.conf.storage.port=9160
# This specifies the keyspace where data is stored.
janusgraphmr.ioformat.conf.storage.cassandra.keyspace=janusgraph
# This defines the indexing backned configuration used while writing data to JanusGraph.
janusgraphmr.ioformat.conf.index.search.backend=elasticsearch
janusgraphmr.ioformat.conf.index.search.hostname=127.0.0.1
# Use the appropriate properties for the backend when using a different storage backend (HBase) or indexing backend (Solr).

#
# Apache Cassandra InputFormat configuration
#
cassandra.input.partitioner.class=org.apache.cassandra.dht.Murmur3Partitioner

#
# SparkGraphComputer Configuration
#
spark.master=spark://127.0.0.1:7077
spark.executor.memory=1g
spark.executor.extraClassPath=/opt/lib/janusgraph/*
spark.serializer=org.apache.spark.serializer.KryoSerializer
spark.kryo.registrator=org.apache.tinkerpop.gremlin.spark.structure.io.gryo.GryoRegistrator
```
然后使用Gremlin控制台中的属性文件，如下所示：
```
bin/gremlin.sh

         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----
plugin activated: janusgraph.imports
gremlin> :plugin use tinkerpop.hadoop
==>tinkerpop.hadoop activated
gremlin> :plugin use tinkerpop.spark
==>tinkerpop.spark activated
gremlin> // 1. Open a the graph for OLAP processing reading in from Cassandra 3
gremlin> graph = GraphFactory.open('conf/hadoop-graph/read-cassandra-3.properties')
==>hadoopgraph[cassandra3inputformat->gryooutputformat]
gremlin> // 2. Configure the traversal to run with Spark
gremlin> g = graph.traversal().withComputer(SparkGraphComputer)
==>graphtraversalsource[hadoopgraph[cassandra3inputformat->gryooutputformat], sparkgraphcomputer]
gremlin> // 3. Run some OLAP traversals
gremlin> g.V().count()
......
==>808
gremlin> g.E().count()
......
==> 8046
```
### 37.3 其他顶点程序
Apache TinkerPop提供各种顶点程序。顶点程序在每个顶点上运行，直到达到终止条件或达到固定的迭代次数。由于顶点程序的并行特性，它们可以利用Spark或Giraph等并行计算框架来提高性能。

一旦熟悉如何配置JanusGraph以使用Spark，您就可以运行Apache TinkerPop提供的所有其他顶点程序，如Page Rank，Bulk Loading和Peer Pressure。有关更多详细信息，请参阅[TinkerPop VertexProgram文档](http://tinkerpop.apache.org/docs/3.3.3/reference/#vertexprogram)。