# 4. 官方配置
一个JanusGraph图数据库集群由一个或多个JanusGraph实例组成。为了打开一个JanusGraph实例，必须提供一个配置指定JanusGraph如何去设置。
一个JanusGraph配置需要指定那些组件可以使用，控制JanusGraph部署和操作的各个方面。并提供一些调优选项来达到JanusGraph集群的最大使用性能。
一个最低的配置，JanusGraph配置必须定义一个持久化引擎也就是JanusGraph存储后端。[第三章，存储后端](https://docs.janusgraph.org/latest/storage-backends.html)列举了所有支持的持久化引擎以及分别如何配置他们。如果需要支持图的高级查询（例如：全文检索，GEO检索或者范围检索），必须增加配置一个索引后端。详细可见[第IV部分，"索引后端"](https://docs.janusgraph.org/latest/index-backends.html)。如果关注查询性能，需要支持缓存。缓存的设置和调优在[第13章 JanusGraph 缓存](https://docs.janusgraph.org/latest/caching.html)有详细描述。

* 4.1 配置示例
以下部分是一些解释如何在一些通用的后端引擎进行配置的示例，索引系统和性能组件，这只涉及一小部分可用的配置选项。参照[第15章 配置指南](https://docs.janusgraph.org/latest/config-ref.html) 有所有选项的配置。

* 4.1.1. Cassandra+Elasticsearch
设置JanusGraph使用Cassandra在本地运行的持久化引擎

