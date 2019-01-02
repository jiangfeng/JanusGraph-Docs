第29章 Apache Lucene
---
>Apache Lucene是一个完全用Java编写的高性能，功能齐全的文本搜索引擎库。它是一种适用于几乎所有需要全文搜索的应用程序的技术，尤其是跨平台搜索。Apache Lucene是一个可供免费下载的开源项目。 
--[Apache Lucence主页](http://lucene.apache.org/)

JanusGraph支持Apache Lucene作为单机嵌入式索引后端。Lucene有一个略微扩展的特性集，与Elasticsearch相比，它在小型应用程序中表现更好，但仅限于单机部署。
### 29.3 Lucene嵌入式配置
对于单机部署，Lucene运行时嵌入了Janusgraph。Janusgraph在内部启动并与Lucene交互。  
要运行嵌入式Lucene，请将以下配置选项添加到图形配置文件中，其中/data/searchindex指定Lucene应存储索引数据的目录：
```
index.search.backend=lucene
index.search.directory=/data/searchindex
```
在上面的配置中，索引后端被命名search。替换search为其他名称以更改索引的名称。

### 29.2 功能支持
- 全文：支持所有Text谓词以搜索与给定单词，前缀或正则表达式匹配的文本属性。
- Geo：支持Geo谓词以搜索在给定查询几何中相交，包含或包含的地理属性。支持用于索引的点，线和多边形。支持用于查询点属性的圆和框以及用于查询非点属性的所有形状。
- 数字范围：支持所有数字比较Compare。
- 时间：纳秒粒度时间索引。
- 自定义分析器：选择使用自定义分析器
- Not Query-normal-form：支持Query-normal-form（QNF）以外的查询。JanusGraph的QNF是CNF（联合正常形式）的变体，在可能的情况下内联否定。
- 
### 29.3 配置选项
有关所有Lucene特定配置选项的完整列表以及常规JanusGraph配置选项，请参阅[第15章，配置参考](https://docs.janusgraph.org/latest/config-ref.html)。

请注意，每个索引后端选项都需要以index.[INDEX-NAME].where [INDEX-NAME]为索引后端名称的前缀为前缀。例如，如果索引后端名为search，则这些配置选项需要以前缀为前缀index.search.。要配置名为search的索引后端以使用Lucene作为索引系统，请设置以下配置选项：
```
index.search.backend = lucene
```

### 29.4 延伸阅读
有关Lucene的更多信息，请参阅[Apache Lucene主页](http://lucene.apache.org/)和可用文档。