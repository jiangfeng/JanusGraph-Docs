
JanusGraph是一个分布式图形数据库，这意味着它可以在多节点集群中进行设置。 但是，在这样的环境中工作时，有一些重要的事情需要考虑。 此外，如果配置正确，JanusGraph会为用户处理一些特殊注意事项。

## 10.1. 动态图

JanusGraph支持动态创建图形。 这与标准Gremlin Server实现允许访问图形的方式有所不同。 传统上，用户通过相应地配置gremlin-server.yaml文件，在服务器启动时创建与图形的绑定。 例如，如果yaml文件的图形部分如下所示：

	graphs {
		graph1: conf/graph1.properties,
		graph2: conf/graph2.properties
	}


然后，您将使用以下事实访问Gremlin服务器上的图形：字符graph1将根据其提供的属性文件绑定到服务器上打开的图形，对于graph2也是如此。

但是，如果我们使用ConfiguredGraphFactory动态创建图形，那么这些图形由JanusGraphManager管理，图形配置由ConfigurationManagementGraph管理。 这特别有用，因为: 1. 它允许您在服务器启动后定义图形配置，2. 允许在JanusGraph集群中以持久和分布式方式管理图形配置。

要正确使用ConfiguredGraphFactory，必须配置群集中的每个Gremlin Server以使用JanusGraphManager和ConfigurationManagementGraph。 [此处详细说明了此过程](https://docs.janusgraph.org/latest/configuredgraphfactory.html#configuring-JanusGraph-server-for-configuredgraphfactory)。

### 10.1.1. 图一致性参考

如果将所有JanusGraph服务器配置为使用ConfiguredGraphFactory，JanusGraph将确保所有图形表示在群集中的所有JanusGraph节点上都是最新的。

例如，如果在一个JanusGraph节点上更新或删除配置为图形，那么我们必须从集群中每个JanusGraph节点的缓存中逐出该图形。 否则，我们的群集中可能会出现不一致的图形表示。 JanusGraph通过后端系统使用消息日志队列自动处理此驱逐，该系统配置了相关图形。

如果您的某个服务器配置不正确，则可能无法从缓存中成功删除该图表。




> *重要*

> *对TemplateConfiguration的任何更新都不会导致更新先前使用所述模板配置创建的图形配置。如果要更新单个图形配置，则必须使用可用的更新API执行此操作。 然后，这些更新API将导致跨群集中所有JanusGraph节点的graphe缓存逐出。*

### 10.1.1. 动态图和遍历绑定

JanusGraph能够分别在集群中的所有JanusGraph节点上绑定动态创建的图形及其对<graph.graphname>和<graph.graphname> _traversal的遍历引用，最多20秒滞后以使绑定生效在群集中的任何节点上。[在这里阅读更多相关信息](https://docs.janusgraph.org/latest/configuredgraphfactory.html#graph-and-traversal-bindings)。

JanusGraph通过让群集中的每个节点轮询ConfigurationManagementGraph以获取已为其创建配置的所有图形来实现此目的。然后，JanusGraphManager将使用其持久化配置打开所述图形，将其存储在其图形缓存中，并将<graph.graphname>绑定到GremlinExecutor上的图形引用，并将<graph.graphname> _traversal绑定到图形的遍历参考GremlinExecutor。


这允许您在JanusGraph集群中的每个节点上通过字符串绑定访问动态创建的图形及其遍历引用。这对于能够使用Gremlin Server客户端并使用[TinkerPops’s withRemote functionality](https://docs.janusgraph.org/latest/things-to-consider-in-a-multi-node-janusgraph-cluster.html#tinkerpop-with-remote)功能尤为重要。

#### 10.1.2.1. 设置

要设置群集以绑定动态创建的图形及其遍历引用，您必须：

1. 配置每个节点以使用ConfiguredGraphFactory。
2. 配置每个节点使用JanusGraphChannelizer，它将较低级别的Gremlin Server组件（如GremlinExecutor）注入到JanusGraph项目中，使我们能够更好地控制Gremlin Server。
要配置每个节点以使用JanusGraphChannelizer，我们必须更新gremlin-server.yaml来执行此操作：

```
channelizer：org.janusgraph.channelizers.JanusGraphWebSocketChannelizer
```
您可以选择以下几种渠道商：
```
1. org.janusgraph.channelizers.JanusGraphWebSocketChannelizer
2. org.janusgraph.channelizers.JanusGraphHttpChannelizer
3. org.janusgraph.channelizers.JanusGraphNioChannelizer
4. org.janusgraph.channelizers.JanusGraphWsAndHttpChannelizer
```
所有的通道都与TinkerPop对应的功能完全相同。


#### 10.1.2.2 使用TinkerPop’s withRemote功能

由于遍历引用绑定在JanusGraph服务器上，因此我们可以使用TinkerPop的withRemote功能。 这将允许在本地运行gremlin查询，远程图形引用。 传统上，通过发送字符串脚本表示来运行对远程Gremlin服务器的查询，这些表示在远程服务器上处理并且响应被序列化并发回。 但是，TinkerPop还允许使用remoteGraph，如果您正在构建一个可轻松转移到多个实现的TinkerPop兼容图形基础结构，这可能很有用。

要在JanusGraph中使用此功能，我们必须首先确保在远程JanusGraph集群上创建了一个图形：

	ConfiguredGraphFactory.create("graph1");

接下来，我们必须等待20秒，以确保遍历引用绑定在远程集群中的每个JanusGraph节点上。

最后，我们可以在本地使用withRemote方法来访问对远程图的本地引用：

```
gremlin> cluster = Cluster.open('conf/remote-objects.yaml')
==>localhost/127.0.0.1:8182
gremlin> graph = EmptyGraph.instance()
==>emptygraph[empty]
gremlin> g = graph.traversal().withRemote(DriverRemoteConnection.using(cluster, "graph1_traversal"))
==>graphtraversalsource[emptygraph[empty], standard]
```

为了完成，上面的conf/remote-objects.yaml应告诉Cluster API如何访问远程JanusGraph服务器; 例如，它可能看起来像：

```
hosts: [remoteaddress1.com, remoteaddress2.com]
port: 8182
username: admin
password: password
connectionPool: { enableSsl: true }
serializer: { className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0, config: { ioRegistries: [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistry] }}
```




