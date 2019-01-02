**第18章Apache Cassandra**

当您需要可扩展性和高可用性而不影响性能时，Apache Cassandra数据库是正确的选择。商用硬件或云基础架构的线性可扩展性和经过验证的容错使其成为关键任务数据的完美平台。Cassandra对跨多个数据中心进行复制的支持是同类产品中最佳的，可为您的用户提供更低的延迟，并让您在知道自己可以在区域中断后幸存下来。已知最大的Cassandra集群在400多台机器上拥有超过300 TB的数据。
以下部分概述了JanusGraph与Apache Cassandra一起使用的各种方式。

**18.1 Cassandra存储后端**

![modes-local](https://docs.janusgraph.org/latest/images/modes-local.png)

JanusGraph提供以下后端用于Cassandra：

​	•cql - 基于CQL的驱动程序。这是推荐的驱动程序。

​	•cassandrathrift - JanusGraph的Thrift连接池驱动程序

​	•cassandra- Astyanax driver。Astyanax项目已经停止。

​	•embeddedcassandra - 用于在同一个JVM中运行Cassandra和JanusGraph的嵌入式驱动程序

Cassandra有两个供客户使用的协议：CQL和Thrift。Thrift是最初的界面，但是从Cassandra 2.1开始就被弃用了。JanusGraph的核心最初是在Thrift弃用之前编写的，它有几个支持Thrift的类。使用Cassandra 4.0，将在Cassandra中删除Thrift支持。建议JanusGraph用户使用cql存储后端。

   注意：如果您计划使用基于Thrift的驱动程序并且使用的是Cassandra 2.2或更高版本，则需要显式启用Thrift，以便JanusGraph可以连接到群集。通过./bin/nodetool enablethrift在每个Cassandra节点上运行来完成此操作。

   注意：如果在Cassandra上启用了安全性，则用户必须具有CREATE permission on <all keyspaces>，否则必须由管理员提前创建密钥空间，包括所需的表或用户必须具有的密钥空间CREATE permission on <the configured keyspace>。包含所需表的create table文件位于conf/cassandra/cassandraTables.cql。请在执行之前定义键空间。

**18.2. 本地服务器模式**

Cassandra可以作为JanusGraph和最终用户应用程序在同一本地主机上的独立数据库运行。在这个模型中，JanusGraph和Cassandra通过localhost套接字相互通信。在Cassandra上运行JanusGraph需要以下设置步骤：

​	1.下载Cassandra，解压缩，并在conf/cassandra.yaml和中设置文件系统路径conf/log4j-server.properties

​	2.使用预打包发行版中提供的默认配置文件将Gremlin Server连接到Cassandra需要启用Cassandra Thrift。启用Cassandra Thrift打开conf/cassandra.yaml并更新start_rpc: false到start_rpc: true。如果Cassandra已经运行，可以手动启动Thrift bin/nodetool enablethrift。可以通过bin/nodetoolstatusthrift 验证Thrift状态。

​	3.通过调用bin/cassandra -f解压缩Cassandra的目录中的命令行启动Cassandra 。读取输出以检查Cassandra是否已成功启动。

您可以按如下方式创建Cassandra JanusGraph

`JanusGraph g = JanusGraphFactory.build().
set("storage.backend", "cql").
set("storage.hostname", "127.0.0.1").
open();`


在Gremlin控制台中，您无法定义变量的类型conf和g。因此，只需省略类型声明即可。

**18.3.本地集装箱模式**

Cassandra没有Windows或OSX的本机安装。在OSX，Windows或Linux上运行Cassandra的最简单方法之一是使用Docker容器。您可以使用单个Docker命令下载并运行Cassandra 。安装您打算使用的JanusGraph版本支持的版本非常重要。可以在“ 版本”页面上特定版本的“经测试的兼容性”部分下找到兼容版本。该卡珊德拉泊坞窗中心页面可以为可用的版本，使用的指令引用。在下面的命令中，正在设置环境变量以启用Cassandra Thrift -e CASSANDRA_START_RPC=true。可以在此处找到端口的描述。端口9160用于Thrift客户端API。端口9042用于CQL本机客户端。端口7000,7001和7099用于节点间通信。Cassandra 3.11版本是JanusGraph 0.2.0的最新兼容版本，在下面的参考命令中指定。

docker run --name jg-cassandra -d -e CASSANDRA_START_RPC = true -p 9160：9160 \
-p 9042：9042 -p 7199：7199 -p 7001：7001 -p 7000：7000 cassandra：3.11

**18.4。远程服务器模式**

![modes-distributed](https://docs.janusgraph.org/latest/images/modes-distributed.png)

当图形需要超出单个机器的范围时，Cassandra和JanusGraph在逻辑上分成不同的机器。在此模型中，Cassandra集群维护图形表示，并且任意数量的JanusGraph实例都维护对Cassandra集群的基于套接字的读/写访问。最终用户应用程序可以在与JanusGraph相同的JVM中直接与JanusGraph交互。

例如，假设我们有一个正在运行的Cassandra集群，其中一台机器的IP地址为77.77.77.77，然后将JanusGraph与集群连接完成如下（逗号分隔IP地址以引用多台机器）：
`JanusGraph graph = JanusGraphFactory.build().
set("storage.backend", "cql").
set("storage.hostname", "77.77.77.77").
open();`

在Gremlin控制台中，您无法定义变量的类型conf和g。因此，只需省略类型声明即可。

**18.5。使用Gremlin Server的远程服务器模式**

![modes-rexster](https://docs.janusgraph.org/latest/images/modes-rexster.png)

Gremlin Server可以包装在前一小节中定义的每个JanusGraph实例上。通过这种方式，最终用户应用程序不必是基于Java的应用程序，因为它可以作为客户端与Gremlin Server进行通信。这种类型的部署非常适用于多语言体系结构，其中使用不同语言编写的各种组件需要在图形上进行引用和计算。

使用bin/gremlin-server.sh然后在外部Gremlin控制台会话中使用bin/gremlin.sh您可以通过线路发送Gremlin命令：
`:plugin use tinkerpop.server
:remote connect tinkerpop.server conf/remote.yaml
:> g.addV()`

在这种情况下，每个Gremlin Server都将配置为连接到Cassandra集群。以下显示了Gremlin Server配置的图形特定片段。有关完整示例以及有关如何配置服务器的详细信息，请参阅第7章JanusGraph Server。

```
...
图表：{
  g：conf / janusgraph-cql.properties
}
scriptEngines：{
  gremlin-groovy：{
    插件：{org.janusgraph.graphdb.tinkerpop.plugin.JanusGraphGremlinPlugin：{}，
               org.apache.tinkerpop.gremlin.server.jsr223.GremlinServerGremlinPlugin：{}，
               org.apache.tinkerpop.gremlin.tinkergraph.jsr223.TinkerGraphGremlinPlugin：{}，
               org.apache.tinkerpop.gremlin.jsr223.ImportGremlinPlugin：{classImports：[java.lang.Math]，methodImports：[java.lang.Math＃*]}，
               org.apache.tinkerpop.gremlin.jsr223.ScriptFileGremlinPlugin：{files：[scripts / empty-sample.groovy]}}}}
...
```


有关Gremlin Server的更多信息，请参阅Apache TinkerPop文档

**18.6。JanusGraph嵌入式模式**

最后，可以将Cassandra嵌入到JanusGraph中，这意味着JanusGraph和Cassandra在同一个JVM中运行，并通过进程调用而不是通过网络进行通信。这消除了(反)序列化和网络协议开销，因此可以带来相当大的性能改进。在这种部署模式下，JanusGraph内部启动Cassandra守护进程，JanusGraphs不再连接到现有的集群，而是自己的集群。
要在嵌入模式下使用JanusGraph，只需配置embeddedcassandra为存储后端即可。下面列出的配置选项也适用于嵌入式Cassandra。在创建JanusGraph集群时，确保各个节点可以通过Gossip协议相互发现，因此设置一个带有Cassandra嵌入式集群的JanusGraph，就像独立的Cassandra集群一样。在嵌入模式下运行JanusGraph时，使用附加配置选项配置Cassandra yaml文件storage.conf-file，该选项将yaml文件指定为完整URL，例如storage.conf-file = file:///home/cassandra.yaml。
当运行嵌入了JanusGraph和Cassandra的集群时，建议通过Gremlin Server公开JanusGraph，以便应用程序可以远程连接到JanusGraph图数据库并执行查询。
注意，运行带有Cassandra的JanusGraph需要进行GC调整。虽然嵌入式Cassandra可以提供较低延迟的查询应答，但其负载下的GC行为更难以预测。

**18.7。Cassandra特定配置**
有关除常规JanusGraph配置选项外的所有Cassandra特定配置选项的完整列表，请参阅第15章，配置参考。

配置Cassandra时，建议考虑以下Cassandra特定配置选项：

​	•读一致性级别：读取操作的Cassandra一致性级别

​	•write-consistency-level：写操作的Cassandra一致性级别

​	•replication-factor：要使用的复制因子。复制因子越高，图形数据库以数据复制为代价加工故障的能力就越强。应为生产系统覆盖默认值以确保稳健性。建议值为3。只有在最初创建密钥空间时才能设置此复制因子。在现有键空间上，将忽略此值。

​	•thrift.frame_size_mb：thrift用于传输的最大帧大小。检索非常大的结果集时增加此值。仅适用于storage.backend = cassandrathrift

​	•keyspace：用于存储JanusGraph图的键空间的名称。允许多个JanusGraph图在同一个Cassandra集群中共存。

有关Cassandra一致性级别和可接受值的更多信息，请参阅Cassandra Thrift API。通常，更高级别更一致且更健壮但具有更高的延迟。



**18.8。全局图操作**
在Cassandra上的JanusGraph支持全局顶点和边缘迭代。但是，请注意所有这些顶点和/或边将被加载到内存中，这可能会导致OutOfMemoryException。使用第37章，JanusGraph和TinkerPop的Hadoop-Gremlin有效地迭代大图中的所有顶点或边。

**18.9。在Amazon EC2上部署**
 	Amazon Elastic Compute Cloud（Amazon EC2）是一种Web服务，可在云中提供可调整大小的计算容量。它旨在使开发人员更轻松地进行Web规模计算。	 

​       按照以下步骤在EC2上设置Cassandra集群，并在Cassandra上部署JanusGraph。要遵循这些说明，您需要具有已建立的身份验证凭据以及AWS和EC2的一些基本知识的Amazon AWS账户。

**18.9.1。设置Cassandra集群**

​	这些用于配置和启动DataStax Cassandra Community Edition AMI的说明基于DataStax AMI Docs，并侧重于与JanusGraph部署相关的方面。

**18.9.2。设置安全组**

​	•导航到EC2控制台仪表板，然后单击“网络和安全”下的“安全组”。

​	•创建一个新的安全组。单击“入站”。将“创建新规则”下拉菜单设置为“自定义TCP规则”。从源0.0.0.0/0添加端口22的规则。从安全组成员添加端口1024-65535的规则。如果您不想在安全组成员之间打开所有非特权端口，则至少在安全组成员中打开7000,7199和9160。提示：一旦在框中输入“sg”，“Source”下拉列表将自动完成安全组标识符，因此您无需事先准备好准确的值。

**18.9.3。启动DataStax Cassandra AMI**

​	•“ 在所需区域中启动DataStax AMI

​	•在“请求实例向导”的“实例详细信息”页面上，将“实例数”设置为所需的Cassandra节点数。将“实例类型”设置为至少m1.large。我们建议使用m1.large。

​	•在“请求实例向导”的“高级实例选项”页面上，设置“用户数据”下的“作为文本”单选按钮，然后将其填入文本框：

`--clustername [cassandra-cluster-name]
​	--totalnodes [number-of-instances]
​	--version community
​	--opscenter no`
​	
此配置中的[number-of-instances]必须与上一个向导页面上配置的EC2实例数相匹配。[cassandra-cluster-name]可以是用于标识的任何字符串。例如：

​	`--clustername janusgraph
​	--totalnodes 4
​	--version community
​	--opscenter no`

​	在“请求实例向导”的“标签”页面上，您可以应用任何所需的配置。这些标记仅存在于EC2管理级别，对Cassandra守护程序的配置或操作没有影响。

​	在“请求实例向导”的“创建密钥对”页面上，选择现有密钥对或创建新密钥对。包含所选密钥对的私有一半的PEM文件将需要连接到这些实例。

​	在“请求实例向导”的“配置防火墙”页面上，选择先前创建的安全组。

​	在最终向导页面上查看并启动实例。

**18.9.4。验证成功启动实例**

​	•SSH到任何Cassandra实例节点： ssh -i [your-private-key].pem ubuntu@[public-dns-name-of-any-cassandra-instance]

​	•运行Cassandra nodetool nodetool -h 127.0.0.1 ring以检查Cassandra令牌环的状态。您应该在此命令的输出中看到与前面步骤中启动的实例一样多的节点。

​	请注意，AMI需要几分钟来配置每个实例。当您通过SSH连接到实例时，将在成功配置时显示shell提示符。

**18.9.5. 启动JanusGraph实例**
​	启动其他EC2实例以运行JanusGraph，它们在远程服务器模式或远程服务器模式下配置为Gremlin-Server，如上所述。您只需记下其中一个Cassandra集群实例的IP地址，并将其配置为主机名。要运行的特定EC2实例和特定配置取决于您的用例。

**18.9.6. Amazon Linux AMI上的JanusGraph实例示例**

​	•在Cassandra集群的同一区域中启动Amazon Linux AMI。根据所需的资源量选择所需的EC2实例类型。使用默认配置选项，并选择与上一步中配置的Cassandra群集相同的密钥对和安全组。

​	•通过SSH进入新创建的实例ssh -i [your-private-key].pem ec2-user@[public-dns-name-of-the-instance]。您可能需要等待一些实例才能启动。

​	•下载当前的JanusGraph发行版，wget并将本地存档解压缩到主目录。启动Gremlin控制台以验证JanusGraph是否成功运行。有关如何解压缩JanusGraph并启动Gremlin控制台的更多信息，请参阅“ 入门指南”

​	•使用vi janusgraph.properties并添加以下行创建配置文件::

`storage.backend = cql
storage.hostname = [IP-address-of-one-Cassandra-EC2-instance]`

您可以添加此页面或第15章“ 配置参考”中的其他配置选项。
再次启动Gremlin控制台并键入以下内容::

`gremlin> graph = JanusGraphFactory.open（'janusgraph.properties'）
==> janusgraph [CQL：[IP-address-of-one-Cassandra-EC2-instance]`

您已成功将此JanusGraph实例连接到Cassandra集群，并且可以开始在图表上运行。

**18.9.7. 从EC2外部连接到EC2中的Cassandra集群**

在安全组中打开通常的Cassandra端口（9160,7000,7199）是不够的，因为Cassandra节点默认广播他们的ec2内部IP，而不是他们面向公众的IP。

由此产生的行为是，您可以通过连接到任何Cassandra节点上的端口9160在群集上打开JanusGraph图，但是对该图的所有请求都会超时。这是因为Cassandra告诉客户端连接到无法访问的IP。

要解决此问题，请将/etc/cassandra/cassandra.yaml中每个实例的“broadcast-address”属性设置为面向公众的IP，然后重新启动实例。对集群中的所有节点执行此操作。群集返回后，nodetool会报告允许来自本地计算机的连接的正确的面向公众的IP。

更改“广播地址”属性允许您从ec2外部连接到群集，但这也可能意味着源自ec2的流量必须在到达群集之前往返互联网并返回。因此，这种方法仅对开发和测试有用。

