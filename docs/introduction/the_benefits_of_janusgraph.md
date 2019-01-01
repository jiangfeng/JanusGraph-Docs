

JanusGraph设计的目的是处理大图，单机无论是在存储和计算能力上都无法满足大图处理。大规模图实时计算和分析是JanusGraph最基本的优势。本节将讨论更多JanusGraph特有的优势以及它底层支持的存储方案。

## 1.1. JanusGraph的优势

支持非常大的图。JanusGraph通过添加机器横向扩展集群。

支持很大的并发事务处理和图操作处理。通过添加机器横向扩展JanusGraph的事务处理能力，可以在毫秒级别相应大图的复杂查询。

支持使用Hadoop框架进行全局图分析和批量图处理。

支持在很大的图上对顶点和边进行地理位置、数值范围、全文搜索。

原生支持Apache TinkerPop 描述的当前流行的属性图数据模型。

原生支持图遍历语言Gremlin。

通过使用非编程的方式连接很容易与Gremlin Server集成。

提供了很多图级别配置选项用于调节性能。

以顶点为中心的索引提供顶点级查询，以缓解臭名昭着的超级节点问题。

提供优化的磁盘表示，从而允许有效地使用存储和访问速度。

基于 Apache 2 许可协议开放源码。


## 1.2. JanusGraph 使用 Apache Cassandra的优势

![](../img/cassandra-small.svg)

连续可用，没有单点故障。

由于没有主/从架构，因此对图的读/写没有瓶颈。

弹性可扩展性允许加入和移除机器。

缓存层确保内存中多次连续访问的数据可用。

通过添加集群的机器来增加缓存的大小。

可以与 Apache Hadoop集成。

基于 Apache 2 许可协议开放源码。

## 1.3. JanusGraph 使用 HBase的优势

![](../img/hbase_logo.png)

与Apache Hadoop生态系统紧密集成。

原生支持强一致性。

通过添加更多机器进行线性扩展。

严格的一致性读写操作。

方便的基类用于支持Hadoop MapReduce作业操作HBase表。

支持使用JMX导出监控指标。

基于 Apache 2 许可协议开放源码。

## 1.4. JanusGraph 和 CAP 理论

>* 尽管你付出了最大的努力，你的系统仍会遇到很多的错误，以至于必须在减少输出（如：停止响应请求）和降低收获（如：响应不完整的答案）之间做出选择。 此决定应基于业务要求。  -- Coda Hale 

使用数据库时，应充分考虑CAP定理（C =一致性，A =可用性，P =可分区性）。 
JanusGraph发布包中支持3个后端：Apache Cassandra，Apache HBase和Oracle Berkeley DB Java 企业版。 
请注意，BerkeleyDB JE是一个非分布式数据库，通常仅与JanusGraph一起用于测试和探索。

HBase以输出为代价优先考虑一致性，即完成请求的概率。 Cassandra以收获为代价优先考虑可用性，即响应的完整性（数据可用性/完整数据）。
