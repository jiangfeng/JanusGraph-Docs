第28章 Apache Solr
---

Solr是Apache Lucene项目中非常流行的、非常快速的开源企业搜索平台。Solr是一个独立的企业搜索服务器，具有类似rest的API。Solr是高度可靠、可伸缩和容错的，提供分布式索引、复制和负载平衡查询、自动故障转移和恢复、集中式配置等。
 -[Apache Solr主页](http://lucene.apache.org/solr/)
 
JanusGraph支持Apache Solr作为索引后端。以下是JanusGraph支持的一些Solr功能：
全文：支持所有Text谓词以搜索与给定单词，前缀或正则表达式匹配的文本属性。
- Geo：支持所有Geo谓词，以搜索在给定查询几何体中相交，内部，不相交或包含的地理属性。支持用于索引的点，线和多边形。支持圆，框和多边形用于查询点属性和所有形状以查询非点属性。
- 数字范围：支持所有数字比较Compare。
- TTL：支持自动使索引元素过期。
- 时间：毫秒粒度时间索引。
- 自定义分析器：选择使用自定义分析器。

有关Solr将与JanusGraph配合使用的详细信息，请参阅[附录B，版本兼容性](https://docs.janusgraph.org/latest/version-compat.html)。

### 28.1 Solr配置概述
JanusGraph支持Solr在SolrCloud或Solr Standalone（HTTP）配置中运行，以便与混合索引一起使用（请参见第11.1.2节“混合索引”）。通过参数配置所需的连接模式，该参数mode必须设置为cloud或者http，前者是默认值。例如，要明确指定Solr在SolrCloud配置中运行，请将以下属性指定为JanusGraph配置属性：
```
index.search.solr.mode = cloud
```
这些是一些关键的Solr术语：
Core: 单台机器上的单个索引。
Configuration: solrconfig.xml， schema.xml和定义核心所需的其他文件。
Collection: 可以跨越不同计算机上的多个核心的单个逻辑索引。
Configset: 可由多个内核重用的共享配置。

### 28.2 连接到SolrCloud

通过将mode等于设置为连接到SolrCloud集群时，cloud必须指定Zookeeper URL（以及可选的端口），以便JanusGraph可以发现Solr集群并与之交互。
```
index.search.backend=solr
index.search.solr.mode=cloud
index.search.solr.zookeeper-url=localhost:2181
```
与创建新集合相关的许多附加配置选项(仅在SolrCloud操作模式中受支持)可以配置为控制分片行为。有关这些选项的完整列表，请参阅[第15章 配置参考](https://docs.janusgraph.org/latest/config-ref.html)。

SolrCloud利用Zookeeper在Solr服务器之间协调收集和配置信息。使用Zookeeper和SolrCloud可以显著减少使用Solr作为JanusGraph后端索引所需的手动配置量。

#### 28.2.1 Configset配置

创建集合需要configset。configset存储在Zookeeper中，以便允许跨Solr服务器访问它。
- 每个集合在创建时都可以提供自己的configset，以便每个集合可以具有不同的配置。使用此方法，必须手动创建每个集合。
- 如果一个共享的configset将被多个集合重用，那么它可以单独上传到Zookeeper。使用这种方法，JanusGraph可以通过使用共享的configset自动创建集合。另一个好处是重用configset大大减少了存储在Zookeeper中的数据量。

##### 28.2.1.1 使用单个configset
在本例中，使用发行版中找到的Solr的默认JanusGraph配置手动创建名为verticesByAge的集合。创建集合时，使用相同的集合名verticesByAge将配置上传到Zookeeper中。有关可用参数，请参阅[Solr参考指南](https://lucene.apache.org/solr/guide/7_0/kerberos-authentication-plugin.html#define-a-jaas-configuration-file)。
```
# create the collection
$SOLR_HOME/bin/solr create -c verticesByAge -d $JANUSGRAPH_HOME/conf/solr
```
使用JanusGraphManagement相同的集合名称定义混合索引。
```
mgmt = graph.openManagement()
age = mgmt.makePropertyKey("age").dataType(Integer.class).make()
mgmt.buildIndex("verticesByAge", Vertex.class).addKey(age).buildMixedIndex("search")
mgmt.commit()
```
##### 28.2.1.2 使用共享configset
在使用共享configset时，最方便的方法是首先将配置作为一次性操作上载。在本例中，使用发行版中找到的Solr的默认JanusGraph配置，将名为JanusGraph -configset的configset上传到Zookeeper。有关可用参数，请参阅[Solr参考指南](https://lucene.apache.org/solr/guide/6_6/solr-control-script-reference.html#SolrControlScriptReference-CollectionsandCores)。

```
# upload the shared configset into Zookeeper
# Solr 5
$SOLR_HOME/server/scripts/cloud-scripts/zkcli.sh -cmd upconfig -z localhost:2181 \
    -d $JANUSGRAPH_HOME/conf/solr -n janusgraph-configset
# Solr 6 and higher
$SOLR_HOME/bin/solr zk upconfig -d $JANUSGRAPH_HOME/conf/solr -n janusgraph-configset \
    -z localhost:2181
```
为JanusGraph配置SolrCloud索引后端时，请确保使用该index.search.solr.configset属性提供共享configset的名称。
```
index.search.backend=solr
index.search.solr.mode=cloud
index.search.solr.zookeeper-url=localhost:2181
index.search.solr.configset=janusgraph-configset
```
使用JanusGraphManagement和集合名称定义混合索引。
```
mgmt = graph.openManagement()
age = mgmt.makePropertyKey("age").dataType(Integer.class).make()
mgmt.buildIndex("verticesByAge", Vertex.class).addKey(age).buildMixedIndex("search")
mgmt.commit()
```
### 28.3 连接到Solr Standalone(HTTP)
通过设置mode等于http，通过HTTP连接到Solr Standalone时，必须提供Solr实例的单个或URL列表。
```
index.search.backend=solr
index.search.solr.mode=http
index.search.solr.http-urls=http://localhost:8983/solr
```
用于控制最大连接数，连接超时和传输压缩的其他配置选项可用于HTTP模式。有关这些选项的完整列表，请参见[第15章 配置参考](https://docs.janusgraph.org/latest/config-ref.html)。

#### 28.3.1 核心配置
Solr Standalone用于单个实例，它将配置信息保存在文件系统中。必须为每个混合索引手动创建核心。

要创建核心，需要一个core_name和一个configuration目录。有关可用参数，请参阅Solr参考指南。在本例中，使用发行版中找到的Solr的默认JanusGraph配置创建了一个名为verticesByAge的核心。
```
$SOLR_HOME/bin/solr create -c verticesByAge -d $JANUSGRAPH_HOME/conf/solr
```
使用JanusGraphManagement相同的核心名称定义混合索引。
```
mgmt = graph.openManagement()
age = mgmt.makePropertyKey("age").dataType(Integer.class).make()
mgmt.buildIndex("verticesByAge", Vertex.class).addKey(age).buildMixedIndex("search")
mgmt.commit()
```
### 28.4 Kerberos配置
在连接到受Kerberos保护的Solr环境时，必须指定使用的Kerberos，并引用JAAS配置文件来正确配置Solr客户机。在使用Kerberos时，无论Solr以何种模式运行(SolrCloud或Solr独立运行)，都需要这种配置。
```
index.search.solr.kerberos-enabled=true
```
通过设置java.security.auth.login.config的绝对路径来确保提供JAAS配置文件。应使用JVM选项设置此属性。例如，要运行，gremlin.sh您需要JAVA_OPTIONS在运行脚本之前设置环境变量：
```
---
export JAVA_OPTIONS="-Djava.security.auth.login.config=/absolute/path/jaas.conf"
$JANUSGRAPH_HOME/bin/gremlin.sh
---
```
有关JAAS配置文件中所需内容的详细信息，请参阅[Solr参考指南](https://lucene.apache.org/solr/guide/7_0/kerberos-authentication-plugin.html#define-a-jaas-configuration-file)。

### 28.5 Solr架构设计

#### 28.5.1 动态字段定义

默认情况下，JanusGraph使用Solr的动态字段功能来定义所有索引键的字段类型。将属性键添加到Solr支持的混合索引时，这不需要额外配置，并且提供比无模式模式更好的性能。

JanusGraph假设在支持Solr集合的schema.xml文件中定义了以下动态字段标记。请注意，solr schema.xml文件中还需要以下字段的附加xml定义才能使用它们。有关详细信息，请参阅JanusGraph安装中./conf/solr/schema.xml目录中提供的示例schema.xml文件。
```
   <dynamicField name="*_i"    type="int"          indexed="true"  stored="true"/>
   <dynamicField name="*_s"    type="string"       indexed="true"  stored="true" />
   <dynamicField name="*_l"    type="long"         indexed="true"  stored="true"/>
   <dynamicField name="*_t"    type="text_general" indexed="true"  stored="true"/>
   <dynamicField name="*_b"    type="boolean"      indexed="true" stored="true"/>
   <dynamicField name="*_f"    type="float"        indexed="true"  stored="true"/>
   <dynamicField name="*_d"    type="double"       indexed="true"  stored="true"/>
   <dynamicField name="*_g"    type="geo"          indexed="true"  stored="true"/>
   <dynamicField name="*_dt"   type="date"         indexed="true"  stored="true"/>
   <dynamicField name="*_uuid" type="uuid"         indexed="true"  stored="true"/>
```
在JanusGraph的默认配置中，属性键名称不必以类型相应的后缀结束，以便利用Solr的动态字段功能。JanusGraph通过对属性键定义的数字标识符和类型相应的后缀进行编码，从属性键名称生成Solr字段名称。这意味着JanusGraph在幕后使用具有类型相应后缀的合成字段名称，而不管使用JanusGraph的应用程序代码定义和使用的属性键名称。可以通过非默认配置覆盖此字段名称映射。这将在下一节中描述。

#### 28.5.2 手动字段定义
如果用户宁愿手动为集合中的每个索引字段定义字段类型，则需要禁用dyn-fields配置选项。在将属性键添加到索引之前，必须在支持Solr架构中定义每个索引属性键的字段。

在此场景中，建议启用字段映射的显式属性键名，以便修复字段名的显式定义。这可以通过两种方法之一来实现：
- 在向索引添加属性键时，通过提供mapped-name参数配置字段名。。有关更多信息，请参见[第25.1节 单个字段映射](https://docs.janusgraph.org/latest/field-mapping.html#index-local-field-mapping)。
- 通过为Solr索引启用map-name配置选项，该选项将使用属性键名作为Solr中的字段名。。有关更多信息，请参见[第25.2节 全局字段映射](https://docs.janusgraph.org/latest/field-mapping.html#index-global-field-mapping)。

#### 28.5.3. Schemaless Mode
JanusGraph还可以与为 Schemaless  Mode配置的SolrCloud集群进行交互。在这种情况下，应禁用dyn-fields配置选项，因为Solr将从值而不是字段名称推断字段类型。

但请注意，Schemaless Mode仅建议用于原型设计和初始应用程序开发，不建议用于生产用途。

### 28.6 故障排除

#### 28.6.1 集合不存在

在定义可以使用集合的索引之前，必须初始化集合(以及所有必需的配置文件)。有关更多信息，请参见连接到SolrCloud。

使用SolrCloud时，Zookeeper zkCli.sh命令行工具可用于检查加载到Zookeeper中的配置。还要验证默认的JanusGraph配置文件是否已复制到solr下的正确位置，以及复制文件的目录是否正确。

#### 28.6.2 找不到指定的Configset

使用SolrCloud时，需要一个configset来为JanusGraph创建一个混合索引。有关更多信息，请参阅Configset配置。

- 如果使用单个配置集，则必须首先手动创建集合。
- 如果使用共享配置集，则必须首先将配置集上载到Zookeeper。

您可以验证配置集及其配置文件是否位于Zookeeper中/configs。有关其他Zookeeper操作，请参阅“ Solr参考指南”。
```
# verify the configset in Zookeeper
# Solr 5
$SOLR_HOME/server/scripts/cloud-scripts/zkcli.sh -cmd list -z localhost:2181
# Solr 6 and higher
$SOLR_HOME/bin/solr zk ls -r /configs/configset-name -z localhost:2181
```

#### 28.6.3 HTTP 404错误
使用Solr Standalone（HTTP）模式时可能会遇到此错误。错误的一个例子：

```
20:01:22 ERROR org.janusgraph.diskstorage.solr.SolrIndex  - Unable to save documents
to Solr as one of the shape objects stored were not compatible with Solr.
org.apache.solr.client.solrj.impl.HttpSolrClient$RemoteSolrException: Error from server
at http://localhost:8983/solr: Expected mime type application/octet-stream but got text/html.
<html>
<head>
<meta http-equiv="Content-Type" content="text/html;charset=utf-8"/>
<title>Error 404 Not Found</title>
</head>
<body><h2>HTTP ERROR 404</h2>
<p>Problem accessing /solr/verticesByAge/update. Reason:
<pre>    Not Found</pre></p>
</body>
</html>
```
在尝试将数据存储到索引之前，请确保手动创建核心。有关更多信息，请参阅核心配置。

#### 28.6.4 核心或集合名称无效
核心或集合名称是标识符。它必须完全由句号，下划线，连字符和/或字母数字组成，也可能不以连字符开头。

#### 28.6.5 连接问题

无论操作模式如何，Solr实例或Solr实例集群必须正在运行并可从JanusGraph实例访问，以便JanusGraph将Solr用作索引后端。检查Solr集群是否正常运行，并且它是可见的，并且可以通过网络（或本地）从JanusGraph实例访问。

#### 28.6.6 带有地理数据的JTS ClassNotFoundException

Solr依靠Spatial4j进行地理处理。Spatial4j声明了对JTS的可选依赖（“JTS拓扑套件”）。某些地理字段定义和查询功能需要JTS。如果JTS jar不在Solr守护程序的类路径上，并且schema.xml中的字段使用地理类型，则Solr可能会在其中一个缺少的JTS类上抛出ClassNotFoundException。使用旨在与JanusGraph一起使用的schema.xml文件启动Solr时会出现异常，但CREATE在Solr CoreAdmin API中调用时也会出现异常。虽然根本原因是相同的，但客户端和服务器端的格式略有不同。

以下是Solr服务器日志的代表示例：
```
ERROR [http-8983-exec-5] 2014-10-07 02:54:06, 665 SolrCoreResourceManager.java (line 344) com/vividsolutions/jts/geom/Geometry
java.lang.NoClassDefFoundError: com/vividsolutions/jts/geom/Geometry
        at com.spatial4j.core.context.jts.JtsSpatialContextFactory.newSpatialContext(JtsSpatialContextFactory.java:30)
        at com.spatial4j.core.context.SpatialContextFactory.makeSpatialContext(SpatialContextFactory.java:83)
        at org.apache.solr.schema.AbstractSpatialFieldType.init(AbstractSpatialFieldType.java:95)
        at org.apache.solr.schema.AbstractSpatialPrefixTreeFieldType.init(AbstractSpatialPrefixTreeFieldType.java:43)
        at org.apache.solr.schema.SpatialRecursivePrefixTreeFieldType.init(SpatialRecursivePrefixTreeFieldType.java:37)
        at org.apache.solr.schema.FieldType.setArgs(FieldType.java:164)
        at org.apache.solr.schema.FieldTypePluginLoader.init(FieldTypePluginLoader.java:141)
        at org.apache.solr.schema.FieldTypePluginLoader.init(FieldTypePluginLoader.java:43)
        at org.apache.solr.util.plugin.AbstractPluginLoader.load(AbstractPluginLoader.java:190)
        at org.apache.solr.schema.IndexSchema.readSchema(IndexSchema.java:470)
        at com.datastax.bdp.search.solr.CassandraIndexSchema.readSchema(CassandraIndexSchema.java:72)
        at org.apache.solr.schema.IndexSchema.<init>(IndexSchema.java:168)
        at com.datastax.bdp.search.solr.CassandraIndexSchema.<init>(CassandraIndexSchema.java:54)
        at com.datastax.bdp.search.solr.core.CassandraCoreContainer.create(CassandraCoreContainer.java:210)
        at com.datastax.bdp.search.solr.core.SolrCoreResourceManager.createCore(SolrCoreResourceManager.java:256)
        at com.datastax.bdp.search.solr.handler.admin.CassandraCoreAdminHandler.handleCreateAction(CassandraCoreAdminHandler.java:117)
```
以下是通常出现在CREATE向CoreAdmin API 发出相关命令的客户端输出中的内容
```
org.apache.solr.common.SolrException: com/vividsolutions/jts/geom/Geometry
        at com.datastax.bdp.search.solr.core.SolrCoreResourceManager.createCore(SolrCoreResourceManager.java:345)
        at com.datastax.bdp.search.solr.handler.admin.CassandraCoreAdminHandler.handleCreateAction(CassandraCoreAdminHandler.java:117)
        at org.apache.solr.handler.admin.CoreAdminHandler.handleRequestBody(CoreAdminHandler.java:152)
        ...
```
这可以通过将JTS jar添加到JanusGraph和/或Solr服务器的类路径来解决。由于其LGPL许可证，JTS默认不包含在JanusGraph发行版中。用户必须单独下载JTS jar文件并将其复制到JanusGraph和/或Solr服务器lib目录中。如果使用Solr的内置Web服务器，可以将JTS jar复制到示例/solr-webapp/webapp/WEB-INF/ lib目录以将其包含在类路径中。Solr可以重新启动，异常应该消失。首先必须使用正确的schema.xml文件启动Solr，以便存在示例/solr-webapp/webapp/WEB-INF/lib目录。

要确定Solr服务器的理想JTS版本，首先检查Solr集群使用的Spatial4j的版本，然后确定编译Spatial4j版本的JTS版本。Spatial4j 在com.spatial4j:spatial4j工件的pom中声明其目标JTS版本。将JTS jar复制到solr安装中的server/solr-webapp/webapp/WEB-INF/lib目录。

### 28.7 高级Solr配置
#### 28.7.1 DSE搜索
本节介绍JanusGraph与DataStax Enterprise（DSE）搜索的安装和配置。有多种方法可以安装DSE，但本节重点介绍DSE在Linux上的二进制tarball安装选项。本节中的大多数步骤可以推广到DSE的其他安装选项。

按照页面的指示安装DataStax Enterprise使用二进制tarball安装DataStax Enterprise。

导出DSE_HOME并附加到PATH shell环境中。以下是使用Bash语法的示例：

```
export DSE_HOME=/path/to/dse-version.number
export PATH="$DSE_HOME"/bin:"$PATH"
```
为Solr安装JTS。适当的版本因Spatial4j版本而异。从DSE 4.5.2开始，适当的版本是1.13。
```
cd $DSE_HOME/resources/solr/lib
curl -O 'http://central.maven.org/maven2/com/vividsolutions/jts/1.13/jts-1.13.jar'
```
在单个后台守护程序中启动DSE Cassandra和Solr：
```
# The "dse-data" path below was chosen to match the
# "Installing DataStax Enterprise using the binary tarball"
# documentation page from DataStax.  The exact path is not
# significant.
dse cassandra -s -Ddse.solr.data.dir="$DSE_HOME"/dse-data/solr
```
上一个命令会将一些启动信息写入控制台和日志文件路径$DSE_HOME/resources/cassandra/conf/log4j-server.properties的log4j.appender.R.File的配置。

一旦使用Cassandra和Solr的DSE正常启动，请检查群集运行状况nodetool status。单实例环应该显示一个节点带有标志 *U*p 和 *N*ormal:
```
nodetool status
Note: Ownership information does not include topology; for complete information, specify a keyspace
= Datacenter: Solr
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Owns   Host ID                               Token                                    Rack
UN  127.0.0.1  99.89 KB   100.0%  5484ef7b-ebce-4560-80f0-cbdcd9e9f496  -7317038863489909889                     rack1
```
接下来，切换到Gremlin Console并针对DSE实例打开JanusGraph数据库。这将创建JanusGraph的键空间和列族。
```
cd $JANUSGRAPH_HOME
bin/gremlin.sh

         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----
gremlin> graph = JanusGraphFactory.open('conf/janusgraph-cql-solr.properties')
==>janusgraph[cql:[127.0.0.1]]
gremlin> g = graph.traversal()
==>graphtraversalsource[janusgraph[cql:[127.0.0.1]], standard]
gremlin>
```
保持这个Gremlin控制台打开。我们现在休息一下安装一个Solr核心。然后我们将回到此控制台以加载一些示例数据。

接下来，上传JanusGraph的Solr集合的配置文件，然后在DSE中创建核心：
```
# Change to the directory where JanusGraph was extracted.  Later commands
# use relative paths to the Solr config files shipped with the JanusGraph
# distribution.
cd $JANUSGRAPH_HOME

# The name must be URL safe and should contain one dot/full-stop
# character. The part of the name after the dot must not conflict with
# any of JanusGraph's internal CF names.  Starting the part after the dot
# "solr" will avoid a conflict with JanusGraph's internal CF names.
CORE_NAME=janusgraph.solr1
# Where to upload collection configuration and send CoreAdmin requests.
SOLR_HOST=localhost:8983

# The value of index.[X].solr.http-urls in JanusGraph's config file
# should match $SOLR_HOST and $CORE_NAME.  For example, given the
# $CORE_NAME and $SOLR_HOST values above, JanusGraph's config file would
# contain (assuming "search" is the desired index alias):
#
# index.search.solr.http-urls=http://localhost:8983/solr/janusgraph.solr1
#
# The stock JanusGraph config file conf/janusgraph-cql-solr.properties
# ships with this http-urls value.

# Upload Solr config files to DSE Search daemon
for xml in conf/solr/{solrconfig, schema, elevate}.xml ; do
    curl -v http://"$SOLR_HOST"/solr/resource/"$CORE_NAME/$xml" \
      --data-binary @"$xml" -H 'Content-type:text/xml; charset=utf-8'
done
for txt in conf/solr/{protwords, stopwords, synonyms}.txt ; do
    curl -v http://"$SOLR_HOST"/solr/resource/"$CORE_NAME/$txt" \
      --data-binary @"$txt" -H 'Content-type:text/plain; charset=utf-8'
done
sleep 5

# Create core using the Solr config files just uploaded above
curl "http://"$SOLR_HOST"/solr/admin/cores?action=CREATE&name=$CORE_NAME"
sleep 5

# Retrieve and print the status of the core we just created
curl "http://localhost:8983/solr/admin/cores?action=STATUS&core=$CORE_NAME"
```

现在，JanusGraph数据库和支持Solr核心已经可以使用了。我们可以使用God of the Gods 数据集对其进行测试。从上面开始拾取Gremlin控制台会话：
```
// Assuming graph = JanusGraphFactory.open('conf/janusgraph-cql-solr.properties')...
gremlin> GraphOfTheGodsFactory.load(graph)
==>null
```
现在我们可以运行第3章“ 入门”中描述的任何查询。Solr将提供涉及文本和地理谓词的查询。有关JanusGraph和Solr客户端的更详细报告，运行gremlin.sh -l DEBUG并发出一些索引支持的查询。
