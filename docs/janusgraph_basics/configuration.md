

一个JanusGraph图数据库集群由一个或多个JanusGraph实例组成。为了打开一个JanusGraph实例，必须提供一个配置指定JanusGraph如何去设置。

一个JanusGraph配置需要指定那些组件可以使用，控制JanusGraph部署和操作的各个方面。并提供一些调优选项来达到JanusGraph集群的最大使用性能。

一个最低的配置，JanusGraph配置必须定义一个持久化引擎也就是JanusGraph存储后端。[第三章，存储后端](https://docs.janusgraph.org/latest/storage-backends.html)列举了所有支持的持久化引擎以及分别如何配置他们。如果需要支持图的高级查询（例如：全文检索，GEO检索或者范围检索），必须增加配置一个索引后端。详细可见[第IV部分，"索引后端"](https://docs.janusgraph.org/latest/index-backends.html)。如果关注查询性能，需要支持缓存。缓存的设置和调优在[第13章 JanusGraph 缓存](https://docs.janusgraph.org/latest/caching.html)有详细描述。

## 4.1. 配置示例
以下部分是一些解释如何在一些通用的后端引擎进行配置的示例，索引系统和性能组件，这只涉及一小部分可用的配置选项。参照[第15章 配置指南](https://docs.janusgraph.org/latest/config-ref.html) 有所有选项的配置。

### 4.1.1. Cassandra+Elasticsearch
设置JanusGraph使用Cassandra在本地运行的持久化引擎


	storage.backend=cql
	storage.hostname=localhost
 
	index.search.backend=elasticsearch
	index.search.hostname=100.100.101.1, 100.100.101.2
	index.search.elasticsearch.client-only=true

### 4.1.2. HBase+Caching

设置JanusGraph使用远程运行的HBase存储引擎，为了获取更好的性能，同时使用JanusGraph的缓存组件。

	storage.hostname=100.100.101.1
	storage.port=2181
 
	cache.db-cache = true
	cache.db-cache-clean-wait = 20
	cache.db-cache-time = 180000
	cache.db-cache-size = 0.5

### 4.1.3. BerkeleyDB

设置JanusGraph使用BerkeleyDB作为嵌入式存储引擎，将Elasticsearch作为嵌入式索引系统。

	storage.backend=berkeleyje
	storage.directory=/tmp/graph
 
	index.search.backend=elasticsearch
	index.search.directory=/tmp/searchindex
	index.search.elasticsearch.client-only=false

	index.search.elasticsearch.local-mode=true

第15章：配置参考  详细介绍了所有这些配置选项。 JanusGraph发行版的conf目录包含其他配置示例。


### 4.1.4. 更多示例

conf/目录中有几个示例配置文件可用于快速启动JanusGraph。 这些文件的路径可以传递给JanusGraphFactory.open（...），如下所示：

	// Connect to Cassandra on localhost using a default configuration
	graph = JanusGraphFactory.open("conf/janusgraph-cql.properties")
	// Connect to HBase on localhost using a default configuration
	graph = JanusGraphFactory.open("conf/janusgraph-hbase.properties")

## 4.2. 应用配置

如何为JanusGraph提供配置取决于实例化模式。

### 4.2.1 JanusGraph工厂模式

#### 4.2.1.1. Gremlin控制台

JanusGraph发行版包含一个命令行Gremlin Console，它可以让您轻松入门并与JanusGraph进行交互。 
调用bin/gremlin.sh（Unix/Linux或bin/gremlin.ba（Windows)以启动控制台，然后使用出厂打开JanusGraph图，其配置存储在可访问的属性配置文件中：


	graph = JanusGraphFactory.open('path/to/configuration.properties')

#### 4.2.1.2. JanusGraph嵌入


#### 4.2.1.3. 短代码

	如果先前已配置JanusGraph图集群和/或仅需要定义存储后端，则JanusGraphFactory接受存储后端名称和主机名或目录的以冒号分隔的字符串表示形式。

	graph = JanusGraphFactory.open('cql:localhost')

	graph = JanusGraphFactory.open('berkeleyje:/tmp/graph')

### 4.2.2. JanusGraph服务端

JanusGraph本身就是一组没有执行线程的jar文件。 连接和使用JanusGraph数据库有两种基本模式：

模式：

* 1. 可以通过在程序提供执行线程的客户端程序中嵌入JanusGraph调用来使用JanusGraph。
* 2. JanusGraph打包了一个长时间运行的服务器进程，该进程在启动时允许远程客户端或逻辑在单独的程序中运行以进行JanusGraph调用。这个长时间运行的服务器进程称为JanusGraph Server。

对于JanusGraph服务器，JanusGraph使用Apache TinkerPop堆栈的Gremlin Server来为客户端请求提供服务。 JanusGraph提供了一个开箱即用的配置，可以快速启动JanusGraph Server，但可以更改配置以提供广泛的服务器功能。

配置JanusGraph Server是通过位于JanusGraph发行版的./conf/gremlin-server目录中的JanusGraph Server yaml配置文件完成的。要使用图形实例（JanusGraph）配置JanusGraph Server，JanusGraph Server配置文件需要以下设置：
		
	...
	graphs: {
	  graph: conf/janusgraph-berkeleyje.properties
	}
	scriptEngines: {
	  gremlin-groovy: {
	    plugins: { org.janusgraph.graphdb.tinkerpop.plugin.JanusGraphGremlinPlugin: {},
	               org.apache.tinkerpop.gremlin.server.jsr223.GremlinServerGremlinPlugin: {},
	               org.apache.tinkerpop.gremlin.tinkergraph.jsr223.TinkerGraphGremlinPlugin: {},
	               org.apache.tinkerpop.gremlin.jsr223.ImportGremlinPlugin: {classImports: [java.lang.Math], methodImports: [java.lang.Math#*]},
		           org.apache.tinkerpop.gremlin.jsr223.ScriptFileGremlinPlugin: {files: [scripts/empty-sample.groovy]}}}}
	...

图表条目定义了与特定JanusGraph配置的绑定。 在上面的例子中，它将图形绑定到conf / janusgraph-berkeleyje.properties上的JanusGraph配置。 插件条目启用了JanusGraph Gremlin插件，该插件支持自动导入JanusGraph类，以便可以在远程提交的脚本中引用它们。

在第7章JanusGraph Server中了解有关配置和使用JanusGraph Server的更多信息。

#### 4.2.2.1. 分布式服务

JanusGraph zip文件包含一个快速启动服务器组件，有助于更轻松地开始使用Gremlin Server和JanusGraph。 调用bin/janusgraph.sh start以使用Cassandra和Elasticsearch启动Gremlin Server。

>注意 

>出于安全原因，Elasticsearch和janusgraph.sh必须在非root帐户下运行*

## 4.3. 全局配置

JanusGraph区分本地和全局配置选项。本地配置选项适用于单个JanusGraph实例。全局配置选项适用于群集中的所有实例。更具体地说，JanusGraph区分了以下五个配置选项范围：

* LOCAL：这些选项仅适用于单个JanusGraph实例，并在初始化JanusGraph实例时提供的配置中指定。
* MASKABLE：可以通过本地配置文件为单个JanusGraph实例覆盖这些配置选项。如果本地配置文件未指定该选项，则从全局JanusGraph集群配置中读取其值。
* GLOBAL：始终从群集配置中读取这些选项，并且不能在实例的基础上覆盖这些选项。
* GLOBAL_OFFLINE：与GLOBAL一样，但更改这些选项需要重新启动群集以确保整个群集中的值相同。
* FIXED：与GLOBAL一样，但是一旦初始化JanusGraph集群，就无法更改该值。

启动集群中的第一个JanusGraph实例时，将从提供的本地配置文件初始化全局配置选项。随后，通过JanusGraph的管理API完成更改全局配置选项。要访问管理API，请在打开的JanusGraph实例句柄g上调用g.getManagementSystem()。例如，要更改JanusGraph集群上的默认缓存行为：

	mgmt = graph.openManagement()
	mgmt.get('cache.db-cache')
	// Prints the current config setting
	mgmt.set('cache.db-cache', true)
	// Changes option
	mgmt.get('cache.db-cache')
	// Prints 'true'
	mgmt.commit()
	// Changes take effect

### 4.3.1. 离线选项配置

更改配置选项不会影响正在运行的实例，仅适用于新启动的实例。 更改GLOBAL_OFFLINE配置选项需要重新启动集群，以使更改立即对所有实例生效。 要更改GLOBAL_OFFLINE选项，请按以下步骤操作：

* 关闭集群中除一个JanusGraph实例外的所有实例
* 连接到单个实例
* 确保关闭所有正在运行的事务
* 确保没有启动新事务（即群集必须脱机）
* 打开管理API
* 更改配置选项
* 调用commit将自动关闭图形实例
* 重启所有实例

有关更多信息（包括每个选项的配置范围），请参阅第15章“配置参考”中的完整配置选项列表。
