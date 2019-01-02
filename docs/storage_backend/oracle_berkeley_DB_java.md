# **第21章Oracle Berkeley DB Java版**


 Oracle Berkeley DB Java版是一个完全用Java编写的开源，可嵌入，事务存储引擎。它充分利用Java环境来简化开发和部署。Oracle Berkeley DB Java版的体系结构支持读密集型和写密集型工作负载的高性能和并发性。

在[Oracle Berkeley数据库Java版](http://www.oracle.com/technetwork/database/berkeleydb/overview/index-093405.html)存储后端在同一个JVM运行JanusGraph和一台机器上提供本地持久性。因此，BerkeleyDB存储后端要求所有图形数据都适合本地磁盘，并且所有经常访问的图形元素都适合主存储器。这对商品硬件上具有10-100万个顶点的图形施加了实际限制。但是，对于该大小的图形，BerkeleyDB存储后端表现出高性能，因为所有数据都可以在同一JVM中本地访问。

## 21.1. BerkeleyDB JE设置

由于BerkeleyDB在与JanusGraph相同的JVM中运行，因此连接两者只需要简单的配置而无需额外的设置：

```
JanusGraph g = JanusGraphFactory.build().

set("storage.backend", "berkeleyje").

set("storage.directory", "/data/graph").

open();
```

在Gremlin控制台中，您无法定义变量的类型`conf`和`g`。因此，只需省略类型声明即可。

## 21.2. BerkeleyDB特定配置

有关除常规JanusGraph配置选项外的所有BerkeleyDB特定配置选项的完整列表，请参阅[第15章，*配置参考*](https://docs.janusgraph.org/latest/config-ref.html)。

配置BerkeleyDB时，建议考虑以下BerkeleyDB特定配置选项：

·         **transactions**：启用事务并检测冲突的数据库操作。**注意：**如果多个JanusGraph实例与BerkeleyDB的同一实例交互，则禁用事务可以提高性能，但可能导致不一致甚至破坏数据库。

·         **cache-percentage**：要为其缓存分配给BerkeleyDB的JVM堆空间（通过-Xmx配置）的百分比。尽量给BerkeleyDB尽可能多的空间，而不会给JanusGraph带来内存问题。例如，如果JanusGraph仅运行短事务，则使用值80或更高。

## 21.3. 理想的用例

BerkeleyDB存储后端最适合中小尺寸图形，在商用硬件上具有高达1亿个顶点。对于该大小的图形，它可能会提供比分布式存储后端更高的性能。请注意，BerkeleyDB在可以有效处理的并发请求数量方面也受到限制，因为它在单个计算机上运行。因此，它不适合具有许多并发用户变异图形的应用程序，即使该图形是小到中等大小。

由于BerkeleyDB在与JanusGraph相同的JVM中运行，因此该存储后端非常适合使用JanusGraph对应用程序代码进行单元测试。

## 21.4. 全局图操作

由BerkeleyDB支持的JanusGraph支持全局图形操作，例如迭代所有顶点或边。但请注意，此类操作需要扫描整个数据库，这可能需要大量时间来处理较大的图形。

为了不耗尽内存，建议`storage.transactions=false`在迭代大图时禁用transactions（）。启用事务需要BerkeleyDB获取正在读取的数据的读锁定。在遍历整个图形时，这些读取锁定很容易需要比可用内存更多的内存。



 