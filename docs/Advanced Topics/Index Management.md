### 33.1 重建索引
第11.1节“图形索引”和第11.2节“以顶点为中心的索引”描述了如何构建图形全局索引和以顶点为中心的索引来提高查询性能。如果在相同的管理事务中新定义了索引键或标签，则可以立即使用这些索引。在这种情况下，不需要重新索引图，这个部分可以跳过。如果索引键和标签在索引构造之前就已经存在，那么有必要重新索引整个图，以确保索引包含以前添加的元素。本节介绍重建索引过程。

>警告:
重新索引是一个由多个步骤组成的手动过程。必须按正确的顺序仔细遵循这些步骤，以避免索引不一致。

#### 33.1.1 概述
JanusGraph可以在定义索引之后立即开始编写增量索引更新。但是，在索引完成并可用之前，JanusGraph还必须一次性读取与新索引模式类型关联的所有现有图形元素。一旦重新索引作业完成，索引就被完全填充并准备好使用。然后必须启用索引才能在查询处理期间使用。

#### 33.1.2 在重建索引之前
重建索引过程的起点是构建一个索引。有关全局图和顶点中心索引的完整讨论，请参阅第11章，索引以获得更好的性能。请注意，全局图索引由其名称唯一标识。以顶点为中心的索引由其名称和定义索引的边缘标签或属性键的组合惟一标识 —— 后者的名称在本节中称为索引类型，仅适用于以顶点为中心的索引。

在针对现有模式元素构建新索引之后，建议等待几分钟将索引通知到集群。注意索引名(对于以顶点为中心的索引，以及索引类型)，因为在索引检索时需要此信息。

#### 33.1.3 准备重新索引
可以在重新索引作业的两个执行框架之间进行选择：
- MapReduce
- JanusGraphManagement

MapReduce上的Reindex支持大型水平分布式数据库。JanusGraphManagement上的重新索引会生成单机OLAP作业。这是为了方便和快速地在那些小到足以由一台机器处理的数据库。

重新索引需要：

- 索引名称（一个字符串 - 用户在构建新索引时将此提供给JanusGraph）
- 索引类型（字符串 - 构建以顶点为中心的索引的边标签或属性键的名称）。这仅适用于以顶点为中心的索引 - 为全局图索引留空。

#### 33.1.4 在MapReduce上执行重建索引作业
在MapReduce上生成和运行reindex作业的推荐方法是通过MapReduceIndexManagement类。以下是使用此类运行reindex作业的步骤的概述：
- 打开一个JanusGraph实例
- 将图形实例传递给MapReduceIndexManagement构造函数
- 调用updateIndex(<index>, SchemaAction.REINDEX)的MapReduceIndexManagement实例
- 如果尚未启用索引，请启用它 JanusGraphManagement

该类实现了一个updateIndex方法，该方法仅支持其SchemaAction参数的REINDEX和REMOVE_INDEX操作。该类使用Hadoop配置启动Hadoop MapReduce作业，并在类路径上使用jar。支持Hadoop 1和2。该类从给其构造函数的JanusGraph实例中获取关于索引和存储后端(例如Cassandra partitioner)的元数据。
```
graph = JanusGraphFactory.open(...)
mgmt = graph.openManagement()
mr = new MapReduceIndexManagement(graph)
mr.updateIndex(mgmt.getRelationIndex(mgmt.getRelationType("battled"), "battlesByTime"), SchemaAction.REINDEX).get()
mgmt.commit()
```
##### 33.1.4.1 MapReduce上的重建索引示例
以下Gremlin片段在一个自包含示例中使用针对Cassandra存储后端的最小虚拟数据概述了MapReduce reindex过程的所有步骤。
```
// Open a graph
graph = JanusGraphFactory.open("conf/janusgraph-cql-es.properties")
g = graph.traversal()

// Define a property
mgmt = graph.openManagement()
desc = mgmt.makePropertyKey("desc").dataType(String.class).make()
mgmt.commit()

// Insert some data
graph.addVertex("desc", "foo bar")
graph.addVertex("desc", "foo baz")
graph.tx().commit()

// Run a query -- note the planner warning recommending the use of an index
g.V().has("desc", containsText("baz"))

// Create an index
mgmt = graph.openManagement()

desc = mgmt.getPropertyKey("desc")
mixedIndex = mgmt.buildIndex("mixedExample", Vertex.class).addKey(desc).buildMixedIndex("search")
mgmt.commit()

// Rollback or commit transactions on the graph which predate the index definition
graph.tx().rollback()

// Block until the SchemaStatus transitions from INSTALLED to REGISTERED
report = ManagementSystem.awaitGraphIndexStatus(graph, "mixedExample").call()

// Run a JanusGraph-Hadoop job to reindex
mgmt = graph.openManagement()
mr = new MapReduceIndexManagement(graph)
mr.updateIndex(mgmt.getGraphIndex("mixedExample"), SchemaAction.REINDEX).get()

// Enable the index
mgmt = graph.openManagement()
mgmt.updateIndex(mgmt.getGraphIndex("mixedExample"), SchemaAction.ENABLE_INDEX).get()
mgmt.commit()

// Block until the SchemaStatus is ENABLED
mgmt = graph.openManagement()
report = ManagementSystem.awaitGraphIndexStatus(graph, "mixedExample").status(SchemaStatus.ENABLED).call()
mgmt.rollback()

// Run a query -- JanusGraph will use the new index, no planner warning
g.V().has("desc", containsText("baz"))

// Concerned that JanusGraph could have read cache in that last query, instead of relying on the index?
// Start a new instance to rule out cache hits.  Now we're definitely using the index.
graph.close()
graph = JanusGraphFactory.open("conf/janusgraph-cql-es.properties")
g.V().has("desc", containsText("baz"))
```

#### 33.1.5 在JanusGraphManagement上执行重建索引作业
要在JanusGraphManagement上运行reindex作业，请JanusGraphManagement.updateIndex使用SchemaAction.REINDEX参数进行调用。例如：
```
m = graph.openManagement()
i = m.getGraphIndex('indexName')
m.updateIndex(i, SchemaAction.REINDEX).get()
m.commit()
```

##### 33.1.5.1 JanusGraphManagement的示例
下面将一些示例数据加载到BerkeleyDB支持的JanusGraph数据库中，在事后定义索引，使用JanusGraphManagement重新索引，最后启用并使用索引：

```
import org.janusgraph.graphdb.database.management.ManagementSystem

// Load some data from a file without any predefined schema
graph = JanusGraphFactory.open('conf/janusgraph-berkeleyje.properties')
g = graph.traversal()
m = graph.openManagement()
m.makePropertyKey('name').dataType(String.class).cardinality(Cardinality.LIST).make()
m.makePropertyKey('lang').dataType(String.class).cardinality(Cardinality.LIST).make()
m.makePropertyKey('age').dataType(Integer.class).cardinality(Cardinality.LIST).make()
m.commit()
graph.io(IoCore.gryo()).readGraph('data/tinkerpop-modern.gio')
graph.tx().commit()

// Run a query -- note the planner warning recommending the use of an index
g.V().has('name', 'lop')
graph.tx().rollback()

// Create an index
m = graph.openManagement()
m.buildIndex('names', Vertex.class).addKey(m.getPropertyKey('name')).buildCompositeIndex()
m.commit()
graph.tx().commit()

// Block until the SchemaStatus transitions from INSTALLED to REGISTERED
ManagementSystem.awaitGraphIndexStatus(graph, 'names').status(SchemaStatus.REGISTERED).call()

// Reindex using JanusGraphManagement
m = graph.openManagement()
i = m.getGraphIndex('names')
m.updateIndex(i, SchemaAction.REINDEX)
m.commit()

// Enable the index
ManagementSystem.awaitGraphIndexStatus(graph, 'names').status(SchemaStatus.ENABLED).call()

// Run a query -- JanusGraph will use the new index, no planner warning
g.V().has('name', 'lop')
graph.tx().rollback()

// Concerned that JanusGraph could have read cache in that last query, instead of relying on the index?
// Start a new instance to rule out cache hits.  Now we're definitely using the index.
graph.close()
graph = JanusGraphFactory.open("conf/janusgraph-berkeleyje.properties")
g = graph.traversal()
g.V().has('name', 'lop')
```


### 33.2 删除索引
>警告:
索引删除是一个由多个步骤组成的手动过程。必须按正确的顺序仔细遵循这些步骤，以避免索引不一致。

#### 33.2.1 概述
索引删除分为两个阶段。在第一阶段，一个JanusGraph通过存储后端向所有其他人发信号通知索引将被删除。这会将索引的状态更改为DISABLED。此时，JanusGraph停止使用索引来应答查询并停止逐步更新索引。存储后端中与索引相关的数据仍然存在但被忽略。

第二阶段取决于指数是混合的还是复合的。复合索引可以通过JanusGraph删除。与重建索引一样，可以通过MapReduce或JanusGraphManagement进行删除。但是，混合索引必须在索引后端手动删除;JanusGraph不提供从索引后端删除索引的自动机制。
除模式定义和禁用状态外，索引删除删除与索引关联的所有内容。即使在删除之后，索引的这个模式存根仍然保留，尽管它的存储占用可以忽略并且是固定的。

#### 33.2.2 准备删除索引
如果当前启用了索引，则应首先禁用它。这是通过ManagementSystem。
```
mgmt = graph.openManagement()
rindex = mgmt.getRelationIndex(mgmt.getRelationType("battled"), "battlesByTime")
mgmt.updateIndex(rindex, SchemaAction.DISABLE_INDEX).get()
gindex = mgmt.getGraphIndex("byName")
mgmt.updateIndex(gindex, SchemaAction.DISABLE_INDEX).get()
mgmt.commit()
```
一旦索引上的所有键的状态变为DISABLED，就可以删除索引。ManagementSystem中的实用程序可以自动执行等待DISABLED步骤：
```
ManagementSystem.awaitGraphIndexStatus(graph, 'byName').status(SchemaStatus.DISABLED).call()
```
在复合索引之后DISABLED，可以选择两个执行框架来删除它：
- MapReduce
- JanusGraphManagement

MapReduce上的索引删除支持大型水平分布式数据库。JanusGraphManagement上的索引删除会产生单机OLAP作业。这是为了方便和快速地在那些小到足以由一台机器处理的数据库上。

删除索引需要：
- 索引名称（一个字符串 - 用户在构建新索引时将此提供给JanusGraph）
- 索引类型（字符串 - 构建以顶点为中心的索引的边标签或属性键的名称）。这仅适用于以顶点为中心的索引 - 为全局图索引留空。

如概述中所述，必须从索引后端手动删除混合索引。MapReduce框架和JanusGraphManagement框架都不会从索引后端删除混合后端。

33.2.3 在MapReduce上执行索引删除作业
与重建索引一样，在MapReduce上生成和运行索引删除作业的推荐方法是通过MapReduceIndexManagement类。以下是使用此类运行索引删除作业的步骤的概述：
- 打开一个JanusGraph实例
- 如果尚未禁用索引，请将其禁用 JanusGraphManagement
- 将图形实例传递给MapReduceIndexManagement构造函数
- 调用 updateIndex(<index>, SchemaAction.REMOVE_INDEX)

注释代码示例在下一小节中介绍。

##### 33.2.3.1 MapReduce的示例
```
import org.janusgraph.graphdb.database.management.ManagementSystem

// Load the "Graph of the Gods" sample data
graph = JanusGraphFactory.open('conf/janusgraph-cql-es.properties')
g = graph.traversal()
GraphOfTheGodsFactory.load(graph)

g.V().has('name', 'jupiter')

// Disable the "name" composite index
m = graph.openManagement()
nameIndex = m.getGraphIndex('name')
m.updateIndex(nameIndex, SchemaAction.DISABLE_INDEX).get()
m.commit()
graph.tx().commit()

// Block until the SchemaStatus transitions from INSTALLED to REGISTERED
ManagementSystem.awaitGraphIndexStatus(graph, 'name').status(SchemaStatus.DISABLED).call()

// Delete the index using MapReduceIndexJobs
m = graph.openManagement()
mr = new MapReduceIndexManagement(graph)
future = mr.updateIndex(m.getGraphIndex('name'), SchemaAction.REMOVE_INDEX)
m.commit()
graph.tx().commit()
future.get()

// Index still shows up in management interface as DISABLED -- this is normal
m = graph.openManagement()
idx = m.getGraphIndex('name')
idx.getIndexStatus(m.getPropertyKey('name'))
m.rollback()

// JanusGraph should issue a warning about this query requiring a full scan
g.V().has('name', 'jupiter')
```

#### 33.2.4 在JanusGraphManagement上执行索引删除作业
要在JanusGraphManagement上运行索引删除作业，请JanusGraphManagement.updateIndex使用SchemaAction.REMOVE_INDEX参数进行调用。例如：
```
m = graph.openManagement()
i = m.getGraphIndex('indexName')
m.updateIndex(i, SchemaAction.REMOVE_INDEX).get()
m.commit()
```
##### 33.2.4.1 JanusGraphManagement的示例
以下将一些索引样本数据加载到BerkeleyDB支持的JanusGraph数据库中，然后通过JanusGraphManagement禁用和删除索引：
```
import org.janusgraph.graphdb.database.management.ManagementSystem

// Load the "Graph of the Gods" sample data
graph = JanusGraphFactory.open('conf/janusgraph-cql-es.properties')
g = graph.traversal()
GraphOfTheGodsFactory.load(graph)

g.V().has('name', 'jupiter')

// Disable the "name" composite index
m = graph.openManagement()
nameIndex = m.getGraphIndex('name')
m.updateIndex(nameIndex, SchemaAction.DISABLE_INDEX).get()
m.commit()
graph.tx().commit()

// Block until the SchemaStatus transitions from INSTALLED to REGISTERED
ManagementSystem.awaitGraphIndexStatus(graph, 'name').status(SchemaStatus.DISABLED).call()

// Delete the index using JanusGraphManagement
m = graph.openManagement()
nameIndex = m.getGraphIndex('name')
future = m.updateIndex(nameIndex, SchemaAction.REMOVE_INDEX)
m.commit()
graph.tx().commit()

future.get()

m = graph.openManagement()
nameIndex = m.getGraphIndex('name')

g.V().has('name', 'jupiter')
```

### 33.3 索引管理的常见问题
#### 33.3.1 启动作业时出现IllegalArgumentException
在构建索引后不久启动重建索引作业时，作业可能会失败，并出现例如以下情况之一的异常：
```
The index mixedExample is in an invalid state and cannot be indexed.
The following index keys have invalid status: desc has status INSTALLED
(status must be one of [REGISTERED, ENABLED])
```
```
The index mixedExample is in an invalid state and cannot be indexed.
The index has status INSTALLED, but one of [REGISTERED, ENABLED] is required
```

在构建索引时，它的存在被广播到集群中的所有其他JanusGraph实例。这些必须确认索引的存在，然后才能开始重建过程。根据集群的大小和连接速度，确认可能需要一段时间。因此，应该在构建索引之后等待几分钟，然后开始重新索引过程。

请注意，由于JanusGraph实例失败，确认可能会失败。换句话说，群集可能无限期地等待确认失败的实例。在这种情况下，用户必须手动从群集注册表中删除失败的实例，如第32章“ 故障和恢复”中所述。恢复集群状态后，必须通过在管理系统中再次手动注册索引来重新启动确认过程。
```
mgmt = graph.openManagement()
rindex = mgmt.getRelationIndex(mgmt.getRelationType("battled"),"battlesByTime")
mgmt.updateIndex(rindex, SchemaAction.REGISTER_INDEX).get()
gindex = mgmt.getGraphIndex("byName")
mgmt.updateIndex(gindex, SchemaAction.REGISTER_INDEX).get()
mgmt.commit()
```
等待几分钟确认到达后，重新索引作业应该成功启动。

#### 33.3.2 找不到索引
在“重新索引”作业中出现的这个异常表明，具有给定名称的索引不存在，或者名称没有正确指定。当重新索引全局图索引时，只需要指定在构建索引时定义的索引名称。在检索全局图索引时，必须在定义以顶点为中心的索引的边缘标签或属性键的名称之外加上索引的名称。

#### 33.3.3 Cassandra Mappers失败“太多打开的文件”
异常堆栈跟踪的结尾可能如下所示：
```
java.net.SocketException: Too many open files
        at java.net.Socket.createImpl(Socket.java:447)
        at java.net.Socket.getImpl(Socket.java:510)
        at java.net.Socket.setSoLinger(Socket.java:988)
        at org.apache.thrift.transport.TSocket.initSocket(TSocket.java:118)
        at org.apache.thrift.transport.TSocket.<init>(TSocket.java:109)
```
在启用虚拟节点的情况下运行Cassandra时，虚拟节点的数量似乎在映射器的数量之下设置了一个下限。对于具有大量数据的集群，Cassandra可能会生成比虚拟节点更多的映射器，但它似乎至少会生成与虚拟节点一样多的映射器，即使集群可能是空的或接近空的。在编写本文时，缺省值是256。

每个映射器打开并快速关闭几个指向Cassandra的套接字。由于Thrift使用SO_LINGER，这些关闭套接字的客户端内核将进入异步TIME_WAIT。任何时候只有少量的套接字是打开的——通常是较低的个位数——但是可能会有许多延迟的套接字在TIME_WAIT中累积。当本地运行重索引作业(不在分布式MapReduce集群上)时，这种累积最明显，因为所有这些客户端TIME_WAIT套接字都驻留在单个客户机机器上，而不是分散在集群中的许多机器上。结合256个映射器的地板，一个重索引作业在执行过程中可以打开数千个套接字。当这些套接字都在同一个客户机上停留在TIME_WAIT中时，它们就有可能到达打开的文件ulimit, ulimit也控制打开的套接字的数量。打开文件的ulimit通常设置为1024。

以下是在单台机器上重建索引时处理“Too many open files”问题的一些建议：
- 减小Cassandra连接池的最大大小。例如，考虑将cassandrathrift存储后端max-active和max-idle选项设置max-total为1，并设置为-1。有关Cassandra存储后端上连接池设置的完整列表，请参见第15章，配置参考。
- 增加nofileulimit。理想值取决于Cassandra数据集的大小和reindex映射器的吞吐量; 如果从1024开始，请尝试更大的数量级：10000。这对于维持延迟的TIME_WAIT套接字是必要的。reindex作业不会尝试一次打开那么多套接字。
- 在多节点MapReduce集群上运行reindex任务以分散套接字负载。