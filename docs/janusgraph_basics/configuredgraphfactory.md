Chapter 9. ConfiguredGraphFactory  
---
可以使用ConfiguredGraphFactory配置JanusGraph服务。ConfiguredGraphFactory是图形的访问点，类似于JanusGraphFactory。这些图形工厂提供了动态管理驻留在服务器上的图形的方法。

#### 9.1 概览
JanusGraphFactory是一个类，它通过在每次访问图形时提供一个配置对象来提供对图形的访问点。

ConfiguredGraphFactory提供了对您之前使用ConfigurationManagementGraph为其创建配置的图形的访问点。它还提供了一个访问点来管理图形配置。

ConfigurationManagementGraph允许您管理图形配置。  

JanusGraphManager是一个跟踪图形引用的内部服务器组件，前提是您的图形被配置为使用它。  

#### 9.2 ConfiguredGraphFactory与JanusGraphFactory  

然而，这两个图形工厂之间有一个重要的区别:  
1、只有将服务器配置为在服务器启动时使用ConfigurationManagementGraph api，才可以使用ConfiguredGraphFactory。  
使用ConfiguredGraphFactory的好处是:  
1、每次打开图形时，您只需要提供一个字符串来访问图形，而不是JanusGraphFactory——它要求您指定访问图形时希望使用的后端信息。  
2、如果配置了分布式存储后端，那么集群中的所有JanusGraph节点都可以使用您的图形配置。  

#### 9.3 ConfiguredGraphFactory如何工作
ConfiguredGraphFactory提供了两个场景下的图形访问点:  

1、您已经使用ConfigurationManagementGraph#createConfiguration为特定的图形对象创建了配置。在这个场景中，您的图使用使用前面的配置打开。
2、您已经使用ConfigurationManagementGraph#createTemplateConfiguration创建了一个模板配置。在这个场景中，我们通过复制模板配置中存储的所有属性并附加相关的graphName属性，为您正在创建的图形创建一个配置，然后根据该特定配置打开图形。

#### 9.4 访问图

您可以使用ConfiguredGraphFactory.create(“graphName”)或ConfiguredGraphFactory.open(“graphName”)。通过阅读下面关于ConfigurationManagementGraph的部分，了解这两个选项之间的区别。  

您还可以通过使用绑定来访问图形。在9.10节“图和遍历绑定”一节了解更多。  

#### 9.5 列出图

ConfiguredGraphFactory.getGraphNames()将返回一组你使用 ConfigurationManagementGraph APIs创建配置的图。  

另一方面，JanusGraphFactory.getGraphNames()返回一组已实例化的图形名称，引用存储在JanusGraphManager中。  

#### 9.6 删除图

ConfiguredGraphFactory.drop("graphName")将删除图形数据库，存储和索引后端中的所有数据。

>important:这是一个不可逆操作，将删除所有图形和索引数据。  
>important:为了确保集群中所有JanusGraph节点之间的所有图形表示都是一致的，这将从集群中每个节点上的JanusGraphManager图形缓存中删除图形，假设每个节点都已正确配置为使用JanusGraphManager。了解更多关于此特性的信息，以及如何配置您的服务器来使用此特性(https://docs.janusgraph.org/latest/things-to-consider-in-a-multi-node-janusgraph-cluster.html#graph-reference-consistency)。

#### 9.7 为ConfiguredGraphFactory配置JanusGraph Server
为了能够使用ConfiguredGraphFactory，必须将服务器配置为使用ConfigurationManagementGraph api。为此，必须在服务器的YAML的图形映射中注入名为“ConfigurationManagementGraph”的图形变量。例如:

```
graphManager: org.janusgraph.graphdb.management.JanusGraphManager
graphs: {
  ConfigurationManagementGraph: conf/JanusGraph-configurationmanagement.properties
}
```
在这个例子中，我们的ConfigurationManagementGraph图将使用存储在conf/JanusGraph-configurationmanagement中的属性进行配置属性，例如:
```
gremlin.graph=org.janusgraph.core.JanusGraphFactory
storage.backend=cql
graph.graphname=ConfigurationManagementGraph
storage.hostname=127.0.0.1
```
假设GremlinServer成功启动，并且成功实例化了ConfigurationManagementGraph，那么ConfigurationManagementGraph Singleton上可用的所有api也将对该图进行操作。此外，这个图将用于访问使用ConfiguredGraphFactory创建/打开图形的配置。

>重点:JanusGraph发行版中包含的pom.xml将此依赖项列为可选项，但ConfiguredGraphFactory使用了JanusGraphManager,它需要声明对org.apache.tinkerpop:gremlin-server的依赖。因此，如果您遇到NoClassDefFoundError错误，那么请确保根据此消息进行更新。  

#### 9.8 ConfigurationManagementGraph
ConfigurationManagementGraph是一个单例对象，它允许您创建/更新/删除配置，您可以使用这些配置来使用ConfiguredGraphFactory访问您的图形。请参阅上面关于配置服务器以启用这些api的介绍。  

>重点：ConfiguredGraphFactory提供了一个访问点来管理由ConfigurationManagementGraph管理的图形配置，因此您可以对相应的ConfiguredGraphFactory静态方法进行操作，而不是对Singleton本身进行操作。例如，您可以使用ConfiguredGraphFactory.removeTemplateConfiguration()而不是ConfiguredGraphFactory.getInstance().removetemplateconfiguration()。

##### 9.8.1  Graph Configurations

ConfigurationManagementGraph单例允许您创建用于打开特定图形的配置，该图形引用graph.graphname配置。例如:
```
map = new HashMap<String, Object>();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
map.put("graph.graphname", "graph1");
ConfiguredGraphFactory.createConfiguration(new MapConfiguration(map));
```
然后您可以在任何JanusGraph节点上使用:
```
ConfiguredGraphFactory.open("graph1");
```
##### 9.8.2 模板配置
ConfigurationManagementGraph还允许您创建一个模板配置，您可以使用该模板使用相同的配置模板创建许多图形。例如:
```
map = new HashMap<String, Object>();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
ConfiguredGraphFactory.createTemplateConfiguration(new MapConfiguration(map));
```
这样做之后，您可以使用模板配置创建图形:

```
ConfiguredGraphFactory.create("graph2");
```
该方法首先为“graph2”创建一个新的配置，方法是复制与模板配置相关的所有属性，并将其存储在这个特定图的配置中。这意味着未来可以在任何JanusGraph节点上通过以下操作访问此图:
```
ConfiguredGraphFactory.open("graph2");
```

##### 9.8.3 更新配置
与JanusGraphFactory和与定义属性 graph.graphname 的配置交互的ConfiguredGraphFactory的所有交互都要通过JanusGraphManager来跟踪在给定JVM上创建的图形引用。可以把它看作一个图形缓存。因为这个原因:

>重点：对图形配置的任何更新都会导致从JanusGraph集群中每个节点上的图形缓存中逐出相关图形，假设每个节点都已正确配置以使用JanusGraphManager。详细了解此功能以及如何配置服务器以使用[此](https://docs.janusgraph.org/latest/things-to-consider-in-a-multi-node-janusgraph-cluster.html#graph-reference-consistency)功能。

由于使用模板配置创建的图形首先使用复制和创建方法为该图形创建配置，这意味着：
>重点：对使用模板配置创建的特定图表的任何更新都不能保证在特定图表上生效，直到：
- 相关配置已删除： ConfiguredGraphFactory.removeConfiguration("graph2");
- 使用模板配置重新创建图表： ConfiguredGraphFactory.create("graph2");

#### 9.8.4 更新示例
1）我们将Cassandra数据迁移到具有新IP地址的新服务器：
```
map = new HashMap();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
map.put("graph.graphname", "graph1");
ConfiguredGraphFactory.createConfiguration(new
MapConfiguration(map));

g1 = ConfiguredGraphFactory.open("graph1");

// Update configuration
map = new HashMap();
map.put("storage.hostname", "10.0.0.1");
ConfiguredGraphFactory.updateConfiguration("graph1",
map);

// We are now guaranteed to use the updated configuration
g1 = ConfiguredGraphFactory.open("graph1");
```
2）我们在设置中添加了一个Elasticsearch节点：

```
map = new HashMap();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
map.put("graph.graphname", "graph1");
ConfiguredGraphFactory.createConfiguration(new
MapConfiguration(map));

g1 = ConfiguredGraphFactory.open("graph1");

// Update configuration
map = new HashMap();
map.put("index.search.backend", "elasticsearch");
map.put("index.search.hostname", "127.0.0.1");
map.put("index.search.elasticsearch.transport-scheme", "http");
ConfiguredGraphFactory.updateConfiguration("graph1",
map);

// We are now guaranteed to use the updated configuration
g1 = ConfiguredGraphFactory.open("graph1");
```
3）更新使用已更新的模板配置创建的图形配置：
```
map = new HashMap();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
ConfiguredGraphFactory.createTemplateConfiguration(new
MapConfiguration(map));

g1 = ConfiguredGraphFactory.create("graph1");

// Update template configuration
map = new HashMap();
map.put("index.search.backend", "elasticsearch");
map.put("index.search.hostname", "127.0.0.1");
map.put("index.search.elasticsearch.transport-scheme", "http");
ConfiguredGraphFactory.updateTemplateConfiguration(new
MapConfiguration(map));

// Remove Configuration
ConfiguredGraphFactory.removeConfiguration("graph1");

// Recreate
ConfiguredGraphFactory.create("graph1");
// Now this graph's configuration is guaranteed to be updated
```

### 9.9  JanusGraphManager
JanusGraphManager是一个符合TinkerPop graphManager规范的单体。
特别是，JanusGraphManager提供：

- 一种协调机制，用于在给定的JanusGraph节点上实例化图引用
- 图形参考跟踪器（或缓存）
您使用该graph.graphname属性创建的任何图形都将通过该图形JanusGraphManager，从而以协调的方式实例化。图表引用也将放在有问题的JVM上的图形缓存中。

因此，使用JVM上已经实例化的graph.graphname属性打开的任何图形都将从图形缓存中检索。

这就是为什么更新配置需要几个步骤来保证正确性的原因。

#### 9.9.1 如何使用JanusGraphManager
当您在配置中定义定义如何访问图形的属性时，可以使用这个新配置选项。包含此属性的所有配置都将导致图形实例化通过JanusGraphManager(上面解释的过程)进行。

为了向后兼容，任何不提供此参数但在服务器上提供的图都是从.yaml文件中的graphs对象中开始的，这些图将通过JanusGraphManager绑定，JanusGraphManager由为该图提供的键表示。例如，如果您的.yaml graphs对象看起来像:
```
graphManager: org.janusgraph.graphdb.management.JanusGraphManager
graphs {
  graph1: conf/graph1.properties,
  graph2: conf/graph2.properties
}
```
但conf/graph1.properties和conf/graph2.properties不包括属性 graph.graphname，那么这些图将被储存在JanusGraphManager，因此在你的gremlin脚本执行所结合graph1和graph2分别。
#### 9.9.2 重点
为方便起见，如果用于打开图形的配置指定graph.graphname但未指定后端的存储目录，tablename或keyspacename，则相关参数将自动设置为值graph.graphname。但是，如果您提供其中一个参数，则该值始终优先。如果您不提供，则默认为配置选项的默认值。

一个特例是storage.root配置选项。这是一个新的配置选项，用于指定将用于需要本地存储目录访问的任何后端的目录库。如果提供此参数，则还必须提供该graph.graphname属性，绝对存储目录将等于graph.graphname附加到属性值的storage.root属性值。

以下是一些示例用例：

1）为我的Cassandra后端创建模板配置，以便使用此配置创建的每个图形获得与提供给工厂的String `<graphName>` 等效的唯一键空间：
```
map = new HashMap();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
ConfiguredGraphFactory.createTemplateConfiguration(new
MapConfiguration(map));

g1 = ConfiguredGraphFactory.create("graph1"); //keyspace === graph1
g2 = ConfiguredGraphFactory.create("graph2"); //keyspace === graph2
g3 = ConfiguredGraphFactory.create("graph3"); //keyspace === graph3
```
2）为我的BerkeleyJE后端创建模板配置，以便使用此配置创建的每个图形获得一个等同于`<storage.root> / <graph.graphname>`的唯一存储目录：
```
map = new HashMap();
map.put("storage.backend", "berkeleyje");
map.put("storage.root", "/data/graphs");
ConfiguredGraphFactory.createTemplateConfiguration(new
MapConfiguration(map));

g1 = ConfiguredGraphFactory.create("graph1"); //storage directory === /data/graphs/graph1
g2 = ConfiguredGraphFactory.create("graph2"); //storage directory === /data/graphs/graph2
g3 = ConfiguredGraphFactory.create("graph3"); //storage directory === /data/graphs/graph3
```

### 9.10 图形和遍历绑定

使用ConfiguredGraphFactory创建的图形通过“graph.graphname”属性绑定到Gremlin服务器上的执行程序上下文，并且图形的遍历引用通过`<graphname> _traversal`绑定到上下文。这意味着，在第一次创建/打开图形后，在后续连接到服务器时，您可以通过`<graphname>`和`<graphname> _traversal`属性访问图形和遍历引用。

详细了解此功能以及如何配置服务器以使用[此](https://docs.janusgraph.org/latest/things-to-consider-in-a-multi-node-janusgraph-cluster.html#dynamic-graph-and-traversal-bindings)功能。
>重点：如果使用Gremlin控制台和会话连接连接到远程Gremlin服务器，则必须重新连接到服务器以绑定变量。对于任何会话的WebSocket连接也是如此。
JanusGraphManager每20秒重新绑定存储在ConfigurationManagementGraph（或您已创建配置的图形）上的每个图形。这意味着使用ConfigredGraphFactory创建的图形的图形和遍历绑定将在所有JanusGraph节点上可用，最多延迟20秒。它还意味着在服务器重新启动后，节点上仍然可以使用绑定。

#### 9.10.1 绑定示例
```
gremlin> :remote connect tinkerpop.server conf/remote.yaml
==>Configured localhost/127.0.0.1:8182
gremlin> :remote console
==>All scripts will now be sent to Gremlin Server - [localhost/127.0.0.1:8182] - type ':remote console' to return to local mode
gremlin> ConfiguredGraphFactory.open("graph1")
==>standardjanusgraph[cassandrathrift:[127.0.0.1]]
gremlin> graph1
==>standardjanusgraph[cassandrathrift:[127.0.0.1]]
gremlin> graph1_traversal
==>graphtraversalsource[standardjanusgraph[cassandrathrift:[127.0.0.1]], standard]
```
### 9.11 例子
建议在创建Configured Graph Factory模板时使用会话连接。如果未使用会话连接，则必须使用分号将配置的图形工厂模板创建作为单行发送到服务器。有关会话的详细信息，请参见[第7.1.1.1节“连接到Gremlin Server”](https://docs.janusgraph.org/latest/server.html#first-example-connecting-gremlin-server)。

```
gremlin> :remote connect tinkerpop.server conf/remote.yaml session
==>Configured localhost/127.0.0.1:8182

gremlin> :remote console
==>All scripts will now be sent to Gremlin Server - [localhost:8182]-[5206cdde-b231-41fa-9e6c-69feac0fe2b2] - type ':remote console' to return to local mode

gremlin> ConfiguredGraphFactory.open("graph");
Please create configuration for this graph using the
ConfigurationManagementGraph API.

gremlin> ConfiguredGraphFactory.create("graph");
Please create a template Configuration using the
ConfigurationManagementGraph API.

gremlin> map = new HashMap();
gremlin> map.put("storage.backend", "cql");
gremlin> map.put("storage.hostname", "127.0.0.1");
gremlin> map.put("GraphName", "graph1");
gremlin> ConfiguredGraphFactory.createConfiguration(new MapConfiguration(map));
Please include in your configuration the property "graph.graphname".

gremlin> map = new HashMap();
gremlin> map.put("storage.backend", "cql");
gremlin> map.put("storage.hostname", "127.0.0.1");
gremlin> map.put("graph.graphname", "graph1");
gremlin> ConfiguredGraphFactory.createConfiguration(new MapConfiguration(map));
==>null

gremlin> ConfiguredGraphFactory.open("graph1").vertices();

gremlin> map = new HashMap(); map.put("storage.backend",
"cql"); map.put("storage.hostname", "127.0.0.1");
gremlin> map.put("graph.graphname", "graph1");
gremlin> ConfiguredGraphFactory.createTemplateConfiguration(new MapConfiguration(map));
Your template configuration may not contain the property
"graph.graphname".

gremlin> map = new HashMap();
gremlin> map.put("storage.backend",
"cql"); map.put("storage.hostname", "127.0.0.1");
gremlin> ConfiguredGraphFactory.createTemplateConfiguration(new MapConfiguration(map));
==>null

// Each graph is now acting in unique keyspaces equivalent to the
graphnames.
gremlin> g1 = ConfiguredGraphFactory.open("graph1");
gremlin> g2 = ConfiguredGraphFactory.create("graph2");
gremlin> g3 = ConfiguredGraphFactory.create("graph3");
gremlin> g2.addVertex();
gremlin> l = [];
gremlin> l << g1.vertices().size();
==>0
gremlin> l << g2.vertices().size();
==>1
gremlin> l << g3.vertices().size();
==>0

// After a graph is created, you must access it using .open()
gremlin> g2 = ConfiguredGraphFactory.create("graph2"); g2.vertices().size();
Configuration for graph "graph2" already exists.

gremlin> g2 = ConfiguredGraphFactory.open("graph2"); g2.vertices().size();
==>1
```
