
JanusGraph使用Gremlin Server引擎作为服务组件来处理和响应客户端查询。 当打包在JanusGraph中时，Gremlin Server被称为JanusGraph Server。

JanusGraph Server必须手动启动才能使用。 JanusGraph Server提供了一种远程执行Gremlin脚本的方法，该脚本针对托管在其中的一个或多个JanusGraph实例。 本节将介绍如何使用WebSocket配置，以及如何配置JanusGraph Server来处理HTTP端交互。

## 7.1. 入门
### 7.1.1. 使用预打包方式分发

JanusGraph release版本默认提供了Cassandra和Elasticsearch的配置，可以直接运行JanusGraph Server，方便用户快速使用JanusGraph Server。 客户端应用程序使用此默认配置可通过自定义的WebSocket协议连接到JanusGraph Server。 有许多使用不同语言开发的客户端支持此协议。 使用WebSocket接口最常用的客户端是Gremlin控制台。 快速启动捆绑包不代表生产安装，而是提供了一种使用JanusGraph Server开发，测试以及了解各个组件一起工作的方法。 使用此默认配置：

* 从Releases页面下载最新版本的janusgraph-$VERSION.zip文件
* 解压并进入janusgraph- $ VERSION目录
* 运行bin / janusgraph.sh start。 这一步将在一个单独的进程中基于Cassandra / ES配置启动Gremlin Server。 

注意出于安全原因，Elasticsearch和janusgraph.sh必须在非root帐户下运行。

    $bin/janusgraph.sh start 
    Forking Cassandra...
    Running `nodetool statusthrift`.. OK (returned exit status 0 and printed string "running").
    Forking Elasticsearch...
    Connecting to Elasticsearch (127.0.0.1:9300)... OK (connected to 127.0.0.1:9300).
    Forking Gremlin-Server...
    Connecting to Gremlin-Server (127.0.0.1:8182)... OK (connected to 127.0.0.1:8182).
    Run gremlin.sh to connect.

### 7.1.1.1. 连接到Gremlin Server

运行janusgraph.sh后，Gremlin Server将开始监听WebSocket连接。 测试连接的最简单方法是使用Gremlin Console。

使用bin/gremlin.sh启动Gremlin控制台并使用 :remote 和 :> 向Gremlin Server发送Gremlin命令：

    $  bin/gremlin.sh
             \,,,/
             (o o)
    -----oOOo-(3)-oOOo-----
    plugin activated: tinkerpop.server
    plugin activated: tinkerpop.hadoop
    plugin activated: tinkerpop.utilities
    plugin activated: janusgraph.imports
    plugin activated: tinkerpop.tinkergraph
    gremlin> :remote connect tinkerpop.server conf/remote.yaml
    ==>Connected - localhost/127.0.0.1:8182
    gremlin> :> graph.addVertex("name", "stephen")
    ==>v[256]
    gremlin> :> g.V().values('name')
    ==>stephen

:remote命令告诉控制台使用conf/remote.yaml文件远程连接到Gremlin Server。 该文件指向在本地运行的Gremlin Server实例。 :>是“submit”命令，它将该行的Gremlin发送到当前运行中的远端。 默认情况下，远程连接是无会话的，这意味着在控制台中发送的每一行都被解释为单个请求。 可以使用分号作为分隔符在一行上发送多个语句。 或者，你可以通过在创建连接时指定会话来建立具有会话的控制台。 控制台会话允许您跨多行复用变量输入。

    gremlin> :remote connect tinkerpop.server conf/remote.yaml
    ==>Configured localhost/127.0.0.1:8182
    gremlin> graph
    ==>standardjanusgraph[cql:[127.0.0.1]]
    gremlin> g
    ==>graphtraversalsource[standardjanusgraph[cql:[127.0.0.1]], standard]
    gremlin> g.V()
    gremlin> user = "Chris"
    ==>Chris
    gremlin> graph.addVertex("name", user)
    No such property: user for class: Script21
    Type ':help' or ':h' for help.
    Display stack trace? [yN]
    gremlin> :remote connect tinkerpop.server conf/remote.yaml session
    ==>Configured localhost/127.0.0.1:8182-[9acf239e-a3ed-4301-b33f-55c911e04052]
    gremlin> g.V()
    gremlin> user = "Chris"
    ==>Chris
    gremlin> user
    ==>Chris
    gremlin> graph.addVertex("name", user)
    ==>v[4344]
    gremlin> g.V().values('name')
    ==>Chris

## 7.2. 预安装包使用后清理

如果您想重新开始并删除数据库和日志，可以使用janusgraph.sh的clean命令。 在运行clean操作之前要停止服务器。

    $ cd /Path/to/janusgraph/janusgraph-0.2.0-hadoop2/
    $ ./bin/janusgraph.sh stop
    Killing Gremlin-Server (pid 91505)...
    Killing Elasticsearch (pid 91402)...
    Killing Cassandra (pid 91219)...
    $ ./bin/janusgraph.sh clean
    Are you sure you want to delete all stored data and logs? [y/N] y
    Deleted data in /Path/to/janusgraph/janusgraph-0.2.0-hadoop2/db
    Deleted logs in /Path/to/janusgraph/janusgraph-0.2.0-hadoop2/log

## 7.3. 使用WebSocket连接 JanusGraph Server

本章的1节入门部分讲的默认配置就是使用的WebSocket配置。如果要使用自己的Cassandra或HBase环境，需要更改默认配置来启动环境，请按照以下操作步骤：

1. 首先测试本地连接到JanusGraph数据库。无论是使用Gremlin控制台还是用程序测试连接都可以。在JanusGraph的./conf目录下的properties文件中进行适当的更改。例如，编辑./conf/janusgraph-hbase.properties并确保正确指定了storage.backend，storage.hostname和storage.hbase.table等参数。有关为各种后端存储配置JanusGraph的更多信息，请参阅第III部分“后端存储”。确保properties文件包含以下行：
  
    gremlin.graph = org.janusgraph.core.JanusGraphFactory

2. 测试完本地配置连接而且你也配置好了properties文件后，将properties文件从./conf目录复制到./conf/gremlin-server目录。

    cp conf/janusgraph-hbase.properties conf/gremlin-server/socket-janusgraph-hbase-server.properties

3. 复制./conf/gremlin-server/gremlin-server.yaml文件并重新命名为socket-gremlin-server.yaml。如果您需要保留原来的yaml文件，请执行此操作

    cp conf/gremlin-server/gremlin-server.yaml conf/gremlin-server/socket-gremlin-server.yaml

4. 编辑socket-gremlin-server.yaml文件并进行以下更新：

    如果你计划连接到其他的JanusGraph Server而不是本地，需要更新host的IP地址：

    host：10.10.10.100 //服务的IP

    更新graphs配置指向你使用的properties文件，以便JanusGraph Server可以找到并连接到你的JanusGraph实例：

    graphs：{
      graph：conf/gremlin-server/socket-janusgraph-hbase-server.properties
    }

5. 启动JanusGraph服务，并指定刚刚配置的yaml文件：

    bin/gremlin-server.sh ./conf/gremlin-server/socket-gremlin-server.yaml

注意：不能只使用bin/janusgraph.sh。这将使用默认配置启动，从而启动Cassandra / Elasticsearch环境。

6. JanusGraph Server将在WebSocket模式下运行，可以按照第7章的1.1.1节“连接到Gremlin服务”中的内容进行测试。

## 7.4. 使用HTTP连接 JanusGraph Server

第7章1节“入门”中描述的默认配置是WebSocket配置。如果要更改默认配置以使用HTTP连接到JanusGraph Server，请按照下列步骤操作：

首先测试与JanusGraph数据库的本地连接。无论是使用Gremlin控制台还是使用程序测试连接都可以。在JanusGraph的./conf目录中的properties文件中进行适当的更改。例如，编辑./conf/janusgraph-hbase.properties并确保正确指定了storage.backend，storage.hostname和storage.hbase.table参数。有关JanusGraph的更多后端存储配置信息，请参阅第III部分“后端存储”。确保properties文件包含以下行：

    gremlin.graph = org.janusgraph.core.JanusGraphFactory

本地连接测试完成并且你的properties文件确定可用后，将properties文件从./conf目录复制到./conf/gremlin-server目录下。

    cp conf/janusgraph-hbase.properties conf/gremlin-server/http-janusgraph-hbase-server.properties

复制./conf/gremlin-server/gremlin-server.yaml文件更名为http-gremlin-server.yaml。如果你需要参考文件的原始版本，请执行以下命令

    cp conf/gremlin-server/gremlin-server.yaml conf/gremlin-server/http-gremlin-server.yaml

编辑http-gremlin-server.yaml文件并进行以下修改：

如果你打算连接到其他的JanusGraph Server而不是本地服务，请修改host IP：

    host：10.10.10.100

更新channelizer设置来指定HttpChannelizer：

channelizer：org.apache.tinkerpop.gremlin.server.channel.HttpChannelizer

更新graphs部分来指向新的properties文件，以便JanusGraph Server可以找到并连接到你的JanusGraph实例：

    graphs: {
      graph: conf/gremlin-server/http-janusgraph-hbase-server.properties
    }

启动JanusGraph服务器，指向刚刚配置的yaml文件：

    bin/gremlin-server.sh ./conf/gremlin-server/http-gremlin-server.yaml

JanusGraph Server现在应该以HTTP模式运行并可用于测试。 curl可用于验证服务器是否正常工作：

    curl -XPOST -Hcontent-type:application/json -d '{"gremlin":"g.V().count()"}' http://[IP for JanusGraph server host]:8182

## 7.5. 同时使用WebSocket和HTTP连接JanusGraph Server

从JanusGraph 0.2.0开始，你可以配置gremlin-server.yaml以通过同一端口接受WebSocket和HTTP连接。 这可以通过修改如下配置。

    channelizer：org.apache.tinkerpop.gremlin.server.channel.WsAndHttpChannelizer

## 7.6. JanusGraph Server高级配置

### 7.6.1. HTTP身份验证

注意：在以下示例中，credentialsDb应与你正在使用的graph是不同的。 它应该使用合适的后端存储来配置，对于这个后端存储使用不同密钥空间，表或存储目录是合适的。 此graph将通过用户名和密码来使用。

#### 7.6.1.1. HTTP基本身份验证

要在JanusGraph Server中启用基本身份验证，请在gremlin-server.yaml中添加以下配置。

    authentication: {
      authenticator: org.janusgraph.graphdb.tinkerpop.gremlin.server.auth.JanusGraphSimpleAuthenticator,
      authenticationHandler: org.apache.tinkerpop.gremlin.server.handler.HttpBasicAuthenticationHandler,
      config: {
        defaultUsername: user,
        defaultPassword: password,
        credentialsDb: conf/janusgraph-credentials-server.properties
      }
    }

验证是否正确配置了基本身份验证。 例如

    curl -v -XPOST http://localhost:8182 -d '{"gremlin": "g.V().count()"}'

如果身份验证配置正确，则应返回401

    curl -v -XPOST http://localhost:8182 -d '{"gremlin": "g.V().count()"}' -u user:password

如果验证配置正确，则应返回200，结果为4。

### 7.6.2. WebSocket身份验证

WebSocket身份验证通过简单身份验证和安全层（SASL）机制进行。

要启用SASL身份验证，请在gremlin-server.yaml中添加以下配置

    authentication: {
      authenticator: org.janusgraph.graphdb.tinkerpop.gremlin.server.auth.JanusGraphSimpleAuthenticator,
      authenticationHandler: org.apache.tinkerpop.gremlin.server.handler.SaslAuthenticationHandler,
      config: {
        defaultUsername: user,
        defaultPassword: password,
        credentialsDb: conf/janusgraph-credentials-server.properties
      }
    }
    
注意：在上面示例中，credentialsDb应与你正在使用的graph是不同的。 它应该使用合适的后端存储来配置，对于这个后端存储使用不同密钥空间，表或存储目录是合适的。 此graph将通过用户名和密码来使用。

如果你通过gremlin控制台进行连接，则你的remote yaml文件应使用适当的值来配置用户名和密码。

    username: user
    password: password

### 7.6.3. HTTP和WebSocket身份验证

如果你正在使用HTTP和WebSocket组合的方式连接，则可以使用SaslAndHMACAuthenticator进行身份验证，包含WebSocket的SASL，HTTP的基本认证，和HTTP基于哈希的消息身份验证码（HMAC）。 HMAC是基于token的身份验证，通过HTTP使用。 您首先通过/ session端点获取token，然后使用它进行身份验证。它用于节约使用基本身份验证加密密码所花费的时间。

gremlin-server.yaml应添加以下配置

    authentication: {
      authenticator: org.janusgraph.graphdb.tinkerpop.gremlin.server.auth.SaslAndHMACAuthenticator,
      authenticationHandler: org.janusgraph.graphdb.tinkerpop.gremlin.server.handler.SaslAndHMACAuthenticationHandler,
      config: {
        defaultUsername: user,
        defaultPassword: password,
        hmacSecret: secret,
        credentialsDb: conf/janusgraph-credentials-server.properties
      }
    }

注意：在上面示例中，credentialsDb应与你正在使用的graph是不同的。 它应该使用合适的后端存储来配置，对于这个后端存储使用不同密钥空间，表或存储目录是合适的。 此graph将通过用户名和密码来使用。

注意：如果您希望能够在每台服务器上使用相同的HMAC令牌，则在所有正在运行的JanusGraph服务器上应该是相同的。

对于HTTP的HMAC身份验证，这将创建一个/session端点，该端点提供一个token默认情况下在一小时后过期。 token的超时时间可以在authentication.config中的tokenTimeout来配置。 此值为Long值，以毫秒为单位。

你可以使用curl向/ session端点发出get请求来获取令牌。 例如


    curl http://localhost:8182/session -XGET -u user:password
    {"token": "dXNlcjoxNTA5NTQ2NjI0NDUzOkhrclhYaGhRVG9KTnVSRXJ5U2VpdndhalJRcVBtWEpSMzh5WldqRTM4MW89"}

然后，你可以使用”Authorization: Token”的header用于身份验证。 例如


    curl -v http://localhost:8182/session -XPOST -d '{"gremlin": "g.V().count()"}' -H "Authorization: Token dXNlcjoxNTA5NTQ2NjI0NDUzOkhrclhYaGhRVG9KTnVSRXJ5U2VpdndhalJRcVBtWEpSMzh5WldqRTM4MW89"

### 7.6.4. JanusGraph使用TinkerPop Gremlin Server

由于JanusGraph Server是一个包含JanusGraph配置文件的TinkerPop Gremlin Server，因此可以单独下载兼容版本的TinkerPop Gremlin Server并与JanusGraph一起使用。 首先下载适当的Gremlin Server版本，它需要与正在使用的JanusGraph版本(3.3.3)相匹配。

注意：除非特别说明，否则本节中对文件路径的任何引用都是指Gremlin Server的TinkerPop发行版下的路径，而不是带有JanusGraph Server的JanusGraph发行版。

用JanusGraph配置独立的Gremlin Server跟使用打包的JanusGraph Server是类似的。 您应该熟悉graph配置。 基本上，Gremlin Server yaml文件指向特定的图配置文件，这些文件用于实例化它随后将使用的JanusGraph实例。 为了实例化这些Graph实例，Gremlin Server要求在其classpath上提供JanusGraph的相应库和依赖项。

为了演示，这些说明将展示如何在Gremlin Server中为JanusGraph配置BerkeleyDB后端。 如前所述，Gremlin Server需要JanusGraph对其类路径的依赖。 使用正在使用的JanusGraph版本号替换以下命令中的$VERSION：


    bin/gremlin-server.sh -i org.janusgraph janusgraph-all $VERSION

当此过程完成后，Gremlin Server应该具有所有可用的JanusGraph依赖项，因此能够实例化JanusGraph对象。

注意：上面的命令使用Groovy Grape，如果配置不正确，可能会出现下载错误。 有关设置〜/ .groovy / grapeConfig.xml的更多信息，请参阅TinkerPop文档的这一部分。

使用以下内容创建名为GREMLIN_SERVER_HOME/conf/janusgraph.properties的文件：


    gremlin.graph=org.janusgraph.core.JanusGraphFactory
    storage.backend=berkeleyje
    storage.directory=db/berkeley

其他的后端配置也是类似。 请参见第III部分“后端存储”。 如果使用Cassandra，则在janusgraph.properties文件中使用Cassandra配置选项。 唯一保持不变的重要部分是gremlin.graph设置，它应该始终使用JanusGraphFactory。 此设置告诉Gremlin Server如何实例化JanusGraph实例。

接下来创建一个名为GREMLIN_SERVER_HOME/conf/gremlin-server-janusgraph.yaml的文件，其中包含以下内容：

    host: localhost
    port: 8182
    graphs: {
      graph: conf/janusgraph.properties}
    scriptEngines: {
      gremlin-groovy: {
        plugins: { org.janusgraph.graphdb.tinkerpop.plugin.JanusGraphGremlinPlugin: {},
                   org.apache.tinkerpop.gremlin.server.jsr223.GremlinServerGremlinPlugin: {},
                   org.apache.tinkerpop.gremlin.tinkergraph.jsr223.TinkerGraphGremlinPlugin: {},
                   org.apache.tinkerpop.gremlin.jsr223.ImportGremlinPlugin: {classImports: [java.lang.Math], methodImports: [java.lang.Math#*]},
                   org.apache.tinkerpop.gremlin.jsr223.ScriptFileGremlinPlugin: {files: [scripts/empty-sample.groovy]}}}}
    serializers:
      - { className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0, config: { ioRegistries: [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistry] }}
      - { className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0, config: { serializeResultToString: true }}
      - { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV3d0, config: { ioRegistries: [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistry] }}
    metrics: {
    slf4jReporter: {enabled: true, interval: 180000}}

这个配置文件有几个重要的部分，因为它们与JanusGraph有关:

在图中，有一个名为graph的键，其值为conf/janusgraph.properties。 这告诉Gremlin Server实例化一个名为“graph”的Graph实例，并使用conf/janusgraph.properties文件对其进行配置。 “graph”键成为Gremlin Server中Graph实例的唯一名称，可以在提交给它的脚本中引用它。

在插件列表中，有一个对JanusGraphGremlinPlugin的引用，它告诉Gremlin Server初始化“JanusGraph插件”。 “JanusGraph插件”将自动导入JanusGraph特定类，以便在脚本中使用。

请注意脚本键和脚本/janusgraph.groovy的引用。 这个Groovy文件是Gremlin Server和特定ScriptEngine的初始化脚本。 使用以下内容创建scripts/janusgraph.groovy：


def globals = [:]
globals << [g : graph.traversal()]
上面的脚本创建了一个名为globals的Map，并为其分配一个键/值对。 键是g，它的值是从图生成的TraversalSource，它是在配置文件中为Gremlin Server配置的。 此时，现在为Gremlin Server提供的脚本可以使用两个全局变量 - graph和g。

此时，Gremlin Server已配置，可用于连接到新的或现有的JanusGraph数据库。启动server：


    $ bin/gremlin-server.sh conf/gremlin-server-janusgraph.yaml
    [INFO] GremlinServer -
             \,,,/
             (o o)
    -----oOOo-(3)-oOOo-----

    [INFO] GremlinServer - Configuring Gremlin Server from conf/gremlin-server-janusgraph.yaml
    [INFO] MetricManager - Configured Metrics Slf4jReporter configured with interval=180000ms and loggerName=org.apache.tinkerpop.gremlin.server.Settings$Slf4jReporterMetrics
    [INFO] GraphDatabaseConfiguration - Set default timestamp provider MICRO
    [INFO] GraphDatabaseConfiguration - Generated unique-instance-id=7f0000016240-ubuntu1
    [INFO] Backend - Initiated backend operations thread pool of size 8
    [INFO] KCVSLog$MessagePuller - Loaded unidentified ReadMarker start time 2015-10-02T12:28:24.411Z into org.janusgraph.diskstorage.log.kcvs.KCVSLog$MessagePuller@35399441
    [INFO] GraphManager - Graph [graph] was successfully configured via [conf/janusgraph.properties].
    [INFO] ServerGremlinExecutor - Initialized Gremlin thread pool.  Threads in pool named with pattern gremlin-*
    [INFO] ScriptEngines - Loaded gremlin-groovy ScriptEngine
    [INFO] GremlinExecutor - Initialized gremlin-groovy ScriptEngine with scripts/janusgraph.groovy
    [INFO] ServerGremlinExecutor - Initialized GremlinExecutor and configured ScriptEngines.
    [INFO] ServerGremlinExecutor - A GraphTraversalSource is now bound to [g] with graphtraversalsource[standardjanusgraph[berkeleyje:db/berkeley], standard]
    INFO] AbstractChannelizer - Configured application/vnd.gremlin-v3.0+gryo with org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0
    [INFO] AbstractChannelizer - Configured application/vnd.gremlin-v3.0+gryo-stringd with org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0
    [INFO] GremlinServer$1 - Gremlin Server configured with worker thread pool of 1, gremlin pool of 8 and boss thread pool of 1.
    [INFO] GremlinServer$1 - Channel started at port 8182.

以下部分说明如何连接到正在运行的服务器。

#### 7.6.4.1. 通过Gremlin Server连接到JanusGraph
Gremlin Server将在启动时准备好监听WebSocket连接。 测试连接最简单的方法是使用Gremlin Console。

按照第7.1.1.1节“连接到Gremlin服务器”中的说明验证Gremlin服务器是否正常工作

注意：您应该了解的一点是，在使用JanusGraph Server时，Gremlin控制台是从JanusGraph发行版下面启动的，当使用单独的Gremlin Server的测试时，Gremlin控制台是从TinkerPop发行版下启动的。


    GryoMapper mapper = GryoMapper.build().addRegistry(JanusGraphIoRegistry.INSTANCE).create();
    Cluster cluster = Cluster.build().serializer(new GryoMessageSerializerV3d0(mapper)).create();
    Client client = cluster.connect();
    client.submit("g.V()").all().get();

通过将JanusGraphIoRegistry添加到org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0，驱动程序将知道如何正确反序列化JanusGraph返回的自定义数据类型。

# 7.7. JanusGraph Server扩展

通过实现Gremlin Server提供的接口，可以扩展Gremlin Server更多的交互方式，并将其与JanusGraph结合使用。 请参阅相应的TinkerPop文档获取的更多详细信息。

