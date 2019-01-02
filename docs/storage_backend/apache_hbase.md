# **第19章Apache HBase**



![hbase_logo](http://hbase.apache.org/images/hbase_logo.png)

Apache hbase是一个开源的、分布式的、版本化的、非关系数据库。正如BigTable利用Google文件系统提供的分布式数据存储一样，ApacheHBASE在Hadoop和HDFS之上提供了类似BigTable的功能。

## 19.1。HBase设置

以下部分概述了JanusGraph与Apache HBase协同使用的各种方式。

### 19.1.1。本地服务器模式

![modes-local](https://docs.janusgraph.org/latest/images/modes-local.png)

HBase可以作为独立数据库在与JanusGraph和最终用户应用程序相同的本地主机上运行。在这个模型中，JanusGraph和HBase通过`localhost`套接字相互通信。在HBase上运行JanusGraph需要以下设置步骤：

·         从<http://www.apache.org/dyn/closer.cgi/hbase/stable/>下载并提取稳定的HBase 。

·         通过调用解压缩的HBase目录`start-hbase.sh`中的*bin*目录中的脚本来启动HBase 。要停止HBase，请使用`stop-hbase.sh`。

```
$ ./bin/start-hbase.sh
starting master, logging to ../logs/hbase-master-machine-name.local.out
```

now，您可以创建一个HBASE简图，如下所示：

```
JanusGraph graph = JanusGraphFactory.build()
        .set("storage.backend", "hbase")
        .open();
```

请注意，由于默认情况下尝试使用localhost连接，因此您无需指定主机名。此外，在Gremlin控制台中，您无法定义变量的类型`conf`和`g`。因此，只需省略类型声明即可。

### 19.1.2。远程服务器模式

![modes-distributed](https://docs.janusgraph.org/latest/images/modes-distributed.png)

当图形需要超出单个机器的范围时，HBase和JanusGraph在逻辑上分成不同的机器。在此模型中，HBase集群维护图形表示，并且任意数量的JanusGraph实例都维护对HBase集群的基于套接字的读/写访问。最终用户应用程序可以在与JanusGraph相同的JVM中直接与JanusGraph交互。

例如，假设我们有一个运行的HBase集群，其ZooKeeper仲裁由IP地址为77.77.77.77,77.77.77.78和77.77.77.79的三台机器组成，然后将JanusGraph与集群连接如下：

```
JanusGraph g = JanusGraphFactory.build()
        .set("storage.backend", "hbase")
        .set("storage.hostname", "77.77.77.77, 77.77.77.78, 77.77.77.79")
        .open();
```

`storage.hostname`接受以逗号分隔的IP地址列表以及JanusGraph应连接的HBase集群中任何机器子集的主机名。此外，在Gremlin控制台中，您无法定义变量的类型`conf`和`g`。因此，只需省略类型声明即可。

### 19.1.3。使用Gremlin Server的远程服务器模式

![modes-rexster](https://docs.janusgraph.org/latest/images/modes-rexster.png)

最后，Gremlin Server可以包装在前一小节中定义的每个JanusGraph实例上。通过这种方式，最终用户应用程序不必是基于Java的应用程序，因为它可以作为客户端与Gremlin Server进行通信。这种类型的部署非常适用于多语言体系结构，其中使用不同语言编写的各种组件需要在图形上进行引用和计算。

```
http://gremlin-server.janusgraph.machine1/mygraph/vertices/1
http://gremlin-server.janusgraph.machine2/mygraph/tp/gremlin?script=g.v(1).out('follows').out('created')
```

在这种情况下，每个Gremlin Server都将配置为连接到HBase集群。以下显示了Gremlin Server配置的图形特定片段。有关完整示例以及有关如何配置服务器的详细信息[，](https://docs.janusgraph.org/latest/server.html)请参阅[第7章](https://docs.janusgraph.org/latest/server.html)*JanusGraph Server*。

```
...
graphs: {
  g: conf/janusgraph-hbase.properties
}
scriptEngines: {
  gremlin-groovy: {
    plugins: { org.janusgraph.graphdb.tinkerpop.plugin.JanusGraphGremlinPlugin: {},
               org.apache.tinkerpop.gremlin.server.jsr223.GremlinServerGremlinPlugin: {},
               org.apache.tinkerpop.gremlin.tinkergraph.jsr223.TinkerGraphGremlinPlugin: {},
               org.apache.tinkerpop.gremlin.jsr223.ImportGremlinPlugin: {classImports: [java.lang.Math], methodImports: [java.lang.Math#*]},
               org.apache.tinkerpop.gremlin.jsr223.ScriptFileGremlinPlugin: {files: [scripts/empty-sample.groovy]}}}}
...
...
```

## 19.2。HBase特定配置

有关除常规JanusGraph配置选项外的所有HBase特定配置选项的完整列表，请参阅[第15章，*配置参考*](https://docs.janusgraph.org/latest/config-ref.html)。

配置HBase时，建议考虑以下HBase特定配置选项：

·         **storage.hbase.table**：用于存储JanusGraph图的HBase表的名称。允许多个JanusGraph图在同一HBase集群中共存。

有关更多HBase配置选项及其说明，请参阅[HBase配置文档](http://hbase.apache.org/book/config.files.html)。通过`storage.hbase.ext`在JanusGraph配置中为相应的HBase配置选项添加前缀，它将在初始化时传递给HBase。例如，要为HBase使用znode / hbase-secure，请设置属性：`storage.hbase.ext.zookeeper.znode.parent=/hbase-secure`。前缀允许通过JanusGraph配置任意HBase配置选项。

| **重要：**                                                   |
| ------------------------------------------------------------ |
| HBase后端使用毫秒来表示时间戳。在JanusGraph 0.2.0及更早版本中，如果graph.timestamps未明确设置该属性，则默认为MICRO。在这种情况下，graph.timestamps必须将属性显式设置为MILLI。graph.timestamps在任何情况下都不要将该属性设置为其他值。 |

## 19.3。全局图操作

HBus上的JanusGraph支持全局顶点和边缘迭代。但是，请注意所有这些顶点和/或边将被加载到内存中，这可能会导致`OutOfMemoryException`。使用[第37章，*JanusGraph**和TinkerPop**的Hadoop-Gremlin*](https://docs.janusgraph.org/latest/hadoop-tp3.html)地迭代大图中的所有顶点或边。

## 19.4。管理HBase群集的提示和技巧

在[HBase shell](http://wiki.apache.org/hadoop/Hbase/Shell)在主服务器上，可以用于获取集群的整体状态检查。

```
$HBASE_HOME/bin/hbase shell
```

从shell中，以下命令对于理解集群的状态通常是有用的。

```
status 'janusgraph'
status 'simple'
status 'detailed'
```

上面的命令可以识别某个区域服务器是否已经故障。如果是这样，就有可能`ssh`进入失败的区域服务器机器并执行以下操作：

```
sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh stop regionserver
sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh start regionserver
```

使用[PSSH](http://code.google.com/p/parallel-ssh/)可以简化此过程，因为不需要单独登录每台机器来运行命令。将区域服务器的IP地址放入`hosts.txt`文件，然后执行以下操作。

```
pssh -h host.txt sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh stop regionserver
pssh -h host.txt sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh start regionserver
```

接下来，有时需要重新启动主服务器(例如，拒绝连接的异常)。为此，在主服务器上执行以下操作：

```
sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh stop master
sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh start master
```

最后，如果已经部署了HBASE集群，并且主服务器或区域服务器需要更多内存，只需编辑`$HBASE_HOME/conf/hbase-env.sh`相关机器上的文件`-Xmx -Xms`参数。编辑完后，如前面所述，停止/启动主服务器和/或区域服务器。

 