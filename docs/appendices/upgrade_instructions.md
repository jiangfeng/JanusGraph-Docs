### 附录D.升级说明

从Titan或较旧的JanusGraph版本升级时，请按照这些说明进行操作。

### D.1.升级到JanusGraph 0.3.0

```
重要：
您应该在尝试升级之前备份数据！另请注意，升级完成后，您将无法再使用0.3.0之前的客户端版本连接到您的图表。

```

JanusGraph 0.3.0实现了模式约束，这使得有必要引入模式版本的概念。有一个检查可以防止客户端连接,要么使用不同的架构版本，要么没有架构版本的概念。要执行升级

必须在要升级的每个图上设置配置 graph.allow-upgrade=true 选项。必须使用0.3.0或更高版本的JanusGraph打开图表，因为旧版本没有相关概念，graph.storage-version也不允许设置它。

janusgraph.properties文件摘录：



```
# JanusGraph configuration sample: Cassandra over a socket
#
# This file connects to a Cassandra daemon running on localhost via
# Thrift.  Cassandra must already be started before starting JanusGraph
# with this file.

# This option should be removed as soon as the upgrade is complete. Otherwise if this file
# is used in the future to connect to a different graph it could cause an unintended upgrade.
graph.allow-upgrade=true

gremlin.graph=org.janusgraph.core.JanusGraphFactory

# The primary persistence provider used by JanusGraph.  This is required.
# It should be set one of JanusGraph's built-in shorthand names for its
# standard storage backends (shorthands: berkeleyje, cassandrathrift,
# cassandra, astyanax, embeddedcassandra, cql, hbase, inmemory) or to the
# full package and classname of a custom/third-party StoreManager
# implementation.
#
# Default:    (no default value)
# Data Type:  String
# Mutability: LOCAL
storage.backend=cassandrathrift

# The hostname or comma-separated list of hostnames of storage backend
# servers.  This is only applicable to some storage backends, such as
# cassandra and hbase.
#
# Default:    127.0.0.1
# Data Type:  class java.lang.String[]
# Mutability: LOCAL
storage.hostname=127.0.0.1

```

如果graph.allow-upgrade在图形上设置为true graph.storage-version，graph.janusgraph-version则会自动升级以匹配打开图形的服务器或本地客户端的版本级别。您可以验证升级是通过打开管理API和验证的值成功graph.storage-version和graph.janusgraph-version。设置存储版本后，应从graph.allow-upgrade=true属性文件中删除并重新打开图表以确保升级成功。


### D.2.从Titan 1.0.0,1.1.0-SNAPSHOT升级

JanusGraph基于对Titan repotitan11分支 的最新承诺。
JanusGraph对Titan进行了以下更改，因此您需要相应地调整代码和配置：

* 模块名称：titan-*现在janusgraph-*
* 包名：com.thinkaurelius.titan现在org.janusgraph
* 类名：Titan*现在JanusGraph*除了这会复制一个单词的情况，例如，TitanGraph简单JanusGraph而不是 JanusGraphGraph

有关如何配置JanusGraph以读取先前由Titan编写的数据的更多信息，请参阅第39章“ 从Titan迁移”。

###D.3.从JanusGraph 0.1.z升级
###### D.3.1.Elasticsearch

JanusGraph 0.1.z与Elasticsearch 1.5.z兼容。有几种可用的配置选项，包括传输客户端，节点客户端和旧配置跟踪。JanusGraph 0.2.0与1.y到6.y的Elasticsearch版本兼容，但它只使用REST客户端提供单个配置选项。

###### D.3.1.1.TRANSPORT_CLIENT

该TRANSPORT_CLIENT接口已被替换REST_CLIENT。将现有图形迁移到JanusGraph 0.2.0时，interface必须在连接到图形时设置该属性：

```
index.search.backend=elasticsearch
index.search.elasticsearch.interface=REST_CLIENT
index.search.hostname=127.0.0.1
```

连接到图表后，可以通过以下方式进行更改，使属性更新成为永久更新JanusGraphManagement：

```
mgmt = graph.openManagement()
mgmt.set("index.search.elasticsearch.interface", "REST_CLIENT")
mgmt.commit()
```
###### D.3.1.2.节点客户端

可以通过几种方式配置具有JanusGraph的节点客户端。如果节点客户端配置为仅客户端或非数据节点，请按照传输客户端部分中的步骤使用相应的连接到现有群集REST_CLIENT。如果节点客户端是数据节点（本地模式），则将其转换为独立的Elasticsearch节点，该节点在应用程序进程的单独JVM中运行。这可以通过使用JanusGraph配置中的节点配置来启动独立的Elasticsearch 1.5.z节点来完成。例如，我们从这些JanusGraph 0.1.z属性开始：


```
index.search.backend=elasticsearch
index.search.elasticsearch.interface=NODE
index.search.conf-file=es-client.yml
index.search.elasticsearch.ext.node.name=alice
```
配置文件es-client.yml具有属性的位置：

```
node.data: true
path.data: /var/lib/elasticsearch/data
path.work: /var/lib/elasticsearch/work
path.logs: /var/log/elasticsearch
```
可以插入配置文件中es-client.yml的index.search.elasticsearch.ext.*属性和 属性，$ES_HOME/config/elasticsearch.yml 以便可以使用相同的属性启动独立的Elasticsearch 1.5.z节点。请记住，如果任何path位置具有相对路径，则可能需要适当更新这些值。启动独立Elasticsearch节点后，请按照传输客户端 部分中的说明完成到REST_CLIENT界面的迁移。请注意，接口中不使用index.search.conf-file和index.search.elasticsearch.ext.*属性REST_CLIENT，因此可以从配置属性中删除它们。

###### D.3.1.3.旧版配置
JanusGraph 0.1.z不推荐使用旧版配置跟踪，而且JanusGraph 0.2.0不再支持它们。用户应参考前面的部分并迁移到REST_CLIENT。

### D.4.从JanusGraph 0.2.0升级

###### D.4.1.HBase TTL
在JanusGraph 0.2.0中，为HBase存储后端添加了生存时间（TTL）支持。为了利用HBase上的TTL功能，图形时间戳需要为MILLI。如果该graph.timestamps属性未明确设置为MILLI，则默认值为JanusGraph 0.2.0中的MICRO，这会对HBase TTL不起作用。由于graph.timestamps 属性是FIXED，因此需要创建新图表以使graph.timestamps 属性的更改生效。