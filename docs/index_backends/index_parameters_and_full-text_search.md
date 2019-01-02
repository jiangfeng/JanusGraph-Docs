24 索引参数和全文检索
---

定义混合索引时，可以为添加到索引的每个属性键选择性地指定参数列表。
这些参数控制特定键的索引方式。JanusGraph识别以下索引参数。
是否支持这些取决于配置的索引后端。
除了此处列出的参数之外，特定索引后端还可能支持自定义参数。
### 24.1 全文检索

索引字符串值（即具有String.class数据类型的属性键）时，
可以选择将这些值索引为由mapping参数类型控制的文本或字符串。

当该值被索引为文本时，该字符串被标记为一个单词包，
其允许用户有效地查询包含一个或多个单词的所有匹配。
这通常称为全文搜索。当该值被索引为字符串时，
该字符串是索引“as-is”而没有任何进一步的分析或标记化。
这有助于查询精确的字符序列匹配。这通常称为字符串搜索。  

#### 24.1.1 全文检索

默认情况下，字符串被索引为文本。要使此索引选项显式，可以在将属性键索引为文本时定义映射。

```
mgmt = graph.openManagement()
summary = mgmt.makePropertyKey('booksummary').dataType(String.class).make()
mgmt.buildIndex('booksBySummary', Vertex.class).addKey(summary, Mapping.TEXT.asParameter()).buildMixedIndex("search")
mgmt.commit()
```

这与标准的混合索引定义相同，只添加了一个额外的参数来指定索引中的映射 - 在本例中Mapping.TEXT。

当字符串属性被索引为文本时，字符串值被标记化为一包令牌。
确切的标记化取决于索引后端及其配置。JanusGraph的默认标记化将字符串拆分为非字母数字字符，
并删除少于2个字符的任何标记。索引后端使用的标记化可能不同（例如，删除了停用词），
这可能导致在事务内部的修改和索引后端中的已提交数据处理全文搜索查询的方式方面存在细微差别。

当字符串属性被索引为文本时，索引后端仅在图形查询中支持全文搜索谓词。全文搜索不区分大小写。

- textContains：如果（至少）文本字符串中的一个单词与查询字符串匹配，则为true
- textContainsPrefix：如果（至少）文本字符串中的一个单词以查询字符串开头，则为true
- textContainsRegex：如果（至少）文本字符串中的一个单词与给定的正则表达式匹配，则为true
- textContainsFuzzy：如果（至少）文本字符串中的一个单词与查询字符串相似（基于Levenshtein编辑距离），则为true

```
import static org.janusgraph.core.attribute.Text.*
g.V().has('booksummary', textContains('unicorns'))
g.V().has('booksummary', textContainsPrefix('uni'))
g.V().has('booksummary', textContainsRegex('.*corn.*'))
g.V().has('booksummary', textContainsFuzzy('unicorn'))
```
字符串搜索谓词（见下文）可用于查询，但那需要在内存中进行过滤，这可能非常昂贵。

#### 24.1.2 字符串搜索

要将字符串属性索引为字符序列而不进行任何分析或标记化，请将映射指定为Mapping.STRING：
```
mgmt = graph.openManagement()
name = mgmt.makePropertyKey('bookname').dataType(String.class).make()
mgmt.buildIndex('booksBySummary', Vertex.class).addKey(name, Mapping.STRING.asParameter()).buildMixedIndex("search")
mgmt.commit()
```
配置字符串映射后，字符串值将被编入索引，并且可以“as-is”查询 ,包括停用词和非字母字符。但是，在这种情况下，查询必须匹配整个字符串值。
因此，在索引被认为是一个令牌的短字符序列时，字符串映射很有用。

当字符串属性被索引为字符串时，索引后端在图形查询中仅支持以下谓词。字符串搜索区分大小写。

- eq：如果字符串与查询字符串相同
- neq：如果字符串不同于查询字符串
- textPrefix：如果字符串值以给定的查询字符串开头
- textRegex：如果字符串值与给定的正则表达式完全匹配
- textFuzzy：如果字符串值类似于给定的查询字符串（基于Levenshtein编辑距离）
```
import static org.apache.tinkerpop.gremlin.process.traversal.P.*
import static org.janusgraph.core.attribute.Text.*
g.V().has('bookname', eq('unicorns'))
g.V().has('bookname', neq('unicorns'))
g.V().has('bookname', textPrefix('uni'))
g.V().has('bookname', textRegex('.*corn.*'))
g.V().has('bookname', textFuzzy('unicorn'))
```
可以在查询中使用全文搜索谓词，但是那些需要在内存中进行过滤，这可能是非常昂贵的。

#### 24.1.3 全文和字符串搜索
如果您使用Elasticsearch，则可以将属性索引为文本和字符串，从而允许您使用所有谓词进行精确匹配和模糊匹配。
```
mgmt = graph.openManagement()
summary = mgmt.makePropertyKey('booksummary').dataType(String.class).make()
mgmt.buildIndex('booksBySummary', Vertex.class).addKey(summary, Mapping.TEXTSTRING.asParameter()).buildMixedIndex("search")
mgmt.commit()
```

>请注意，数据将存储在索引中两次，一次用于精确匹配，一次用于模糊匹配。
### 24.2 地理映射
默认情况下，JanusGraph支持使用点类型索引地理属性并通过圆或框查询地理属性。
要索引支持按任何地理位置类型查询的非点地理属性，请将映射指定为Mapping.PREFIX_TREE：
```
mgmt = graph.openManagement()
name = mgmt.makePropertyKey('border').dataType(Geoshape.class).make()
mgmt.buildIndex('borderIndex', Vertex.class).addKey(name, Mapping.PREFIX_TREE.asParameter()).buildMixedIndex("search")
mgmt.commit()
```
可以指定其他参数来调整基础前缀树映射的配置。这些可选参数包括前缀树中使用的级别数以及相关的精度。
```
mgmt = graph.openManagement()
name = mgmt.makePropertyKey('border').dataType(Geoshape.class).make()
mgmt.buildIndex('borderIndex', Vertex.class).addKey(name, Mapping.PREFIX_TREE.asParameter(), Parameter.of("index-geo-max-levels", 18), Parameter.of("index-geo-dist-error-pct", 0.0125)).buildMixedIndex("search")
mgmt.commit()
```
请注意，某些索引后端（例如Solr）可能需要额外的外部架构配置来支持和调整索引非点属性。