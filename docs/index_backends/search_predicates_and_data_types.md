23章 搜索谓词和数据类型
---
这个页面列出了所有的比较谓词JanusGraph支持全局图搜索和局部遍历。

### 23.1 比较谓词  
下列比较谓词，枚举了用于索引查询并在上面的示例中使用：  
- eq (equal)
- neq (not equal)
- gt (greater than)
- gte (greater than or equal)
- lt (less than)
- lte (less than or equal)

所有的比较谓词致辞String,numeric,Date和即时的数据类型。
boolean和uuid支持neq和eq

### 23.2 文本谓词

Text枚举指定用于查询匹配文本或字符串值的搜索操作符。两种类型谓词区别：

- 文本搜索谓词在文本字符串被标记化后与文本字符串中的单个单词匹配。这些谓词不区分大小写。

   - textContains：如果（至少）文本字符串中的一个单词与查询字符串匹配，则为true
   - textContainsPrefix：如果（至少）文本字符串中的一个单词以查询字符串开头，则为true
   - textContainsRegex：如果（至少）文本字符串中的一个单词与给定的正则表达式匹配，则为true
   - textContainsFuzzy：如果（至少）文本字符串中的一个单词与查询字符串相似（基于Levenshtein编辑距离），则为true
- 字符串搜索谓词与整个字符串值匹配

  -  textPrefix：如果字符串值以给定的查询字符串开头
  -  textRegex：如果字符串值与给定的正则表达式完全匹配
  -  textFuzzy：如果字符串值类似于给定的查询字符串（基于Levenshtein编辑距离）  

>有关全文和字符串搜索的更多信息，请参见第24.1节“全文搜索”。

### 23.3 地理谓词

下面列举了地理谓词：
- geoIntersect如果两个几何对象具有至少一个共同点（相反geoDisjoint），则这是正确的。
- geoWithin 如果一个几何对象包含另一个几何对象，则成立
- geoDisjoint如果两个几何对象没有共同的点（相反geoIntersect），则这是正确的。
- geoContains 如果一个几何对象被另一个包含，则该方法成立。

>有关地理搜索的详细信息，请参见第24.2节“地理映射”。
 ### 23.4 查询示例
 
 以下查询示例演示了教程上的一些谓词:

```
g.V().has("name", "hercules")
// 2) Find all vertices with an age greater than 50
g.V().has("age", gt(50))
// or find all vertices between 1000 (inclusive) and 5000 (exclusive) years of age and order by increasing age
g.V().has("age", inside(1000, 5000)).order().by("age", incr)
// which returns the same result set as the following query but in reverse order
g.V().has("age", inside(1000, 5000)).order().by("age", decr)
// 3) Find all edges where the place is at most 50 kilometers from the given latitude-longitude pair
g.E().has("place", geoWithin(Geoshape.circle(37.97, 23.72, 50)))
// 4) Find all edges where reason contains the word "loves"
g.E().has("reason", textContains("loves"))
// or all edges which contain two words (need to chunk into individual words)
g.E().has("reason", textContains("loves")).has("reason", textContains("breezes"))
// or all edges which contain words that start with "lov"
g.E().has("reason", textContainsPrefix("lov"))
// or all edges which contain words that match the regular expression "br[ez]*s" in their entirety
g.E().has("reason", textContainsRegex("br[ez]*s"))
// or all edges which contain words similar to "love"
g.E().has("reason", textContainsFuzzy("love"))
// 5) Find all vertices older than a thousand years and named "saturn"
g.V().has("age", gt(1000)).has("name", "saturn")
```

### 23.5 支持数据类型
虽然JanusGraph的复合索引（composite indexes）支持可以存储在JanusGraph中的任何数据类型，
但混合索引（mixed indexes ）仅限于以下数据类型。

- Byte
- Short
- Integer
- Long
- Float
- Double
- String
- Geoshape
- Date
- Instant
- UUID

将来将支持其他数据类型。

### 23.6 地理位置数据类型

Geoshape数据类型支持表示点，圆，框，线，多边形，多点，多线和多边形。索引后端目前支持索引点，圆，框，线，多边形，多点，
多线，多边形和几何集合。仅通过混合索引支持地理空间索引查找。  

要构建Geoshape，请使用以下方法：  

```
//lat, lng
Geoshape.point(37.97, 23.72)
//lat, lng, radius in km
Geoshape.circle(37.97, 23.72, 50)
//SW lat, SW lng, NE lat, NE lng
Geoshape.box(37.97, 23.72, 38.97, 24.72)
//WKT
Geoshape.fromWkt("POLYGON ((35.4 48.9, 35.6 48.9, 35.6 49.1, 35.4 49.1, 35.4 48.9))")
//MultiPoint
Geoshape.geoshape(Geoshape.getShapeFactory().multiPoint().pointXY(60.0, 60.0).pointXY(120.0, 60.0)
  .build())
//MultiLine
Geoshape.geoshape(Geoshape.getShapeFactory().multiLineString()
  .add(Geoshape.getShapeFactory().lineString().pointXY(59.0, 60.0).pointXY(61.0, 60.0))
  .add(Geoshape.getShapeFactory().lineString().pointXY(119.0, 60.0).pointXY(121.0, 60.0)).build())
//MultiPolygon
Geoshape.geoshape(Geoshape.getShapeFactory().multiPolygon()
  .add(Geoshape.getShapeFactory().polygon().pointXY(59.0, 59.0).pointXY(61.0, 59.0)
    .pointXY(61.0, 61.0).pointXY(59.0, 61.0).pointXY(59.0, 59.0))
  .add(Geoshape.getShapeFactory().polygon().pointXY(119.0, 59.0).pointXY(121.0, 59.0)
    .pointXY(121.0, 61.0).pointXY(119.0, 61.0).pointXY(119.0, 59.0)).build())
//GeometryCollection
Geoshape.geoshape(Geoshape.getGeometryCollectionBuilder()
  .add(Geoshape.getShapeFactory().pointXY(60.0, 60.0))
  .add(Geoshape.getShapeFactory().lineString().pointXY(119.0, 60.0).pointXY(121.0, 60.0).build())
  .add(Geoshape.getShapeFactory().polygon().pointXY(119.0, 59.0).pointXY(121.0, 59.0)
    .pointXY(121.0, 61.0).pointXY(119.0, 61.0).pointXY(119.0, 59.0)).build())
```

此外，通过GraphSON导入图形时，几何体可能由GeoJSON表示：
```
//string
"37.97, 23.72"
//list
[37.97, 23.72]
//GeoJSON feature
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [125.6, 10.1]
  },
  "properties": {
    "name": "Dinagat Islands"
  }
}
//GeoJSON geometry
{
  "type": "Point",
  "coordinates": [125.6, 10.1]
}
```
GeoJSON可以指定为Point，Circle，LineString或Polygon。多边形必须关闭。请注意，
与JanusGraph API不同，GeoJSON将坐标指定为lng lat。

### 23.7 集合
如果您使用的是Elasticsearch，则可以使用SET和LIST基数对属性进行索引。例如：
```
mgmt = graph.openManagement()
nameProperty = mgmt.makePropertyKey("names").dataType(String.class).cardinality(Cardinality.SET).make()
mgmt.buildIndex("search", Vertex.class).addKey(nameProperty, Mapping.STRING.asParameter()).buildMixedIndex("search")
mgmt.commit()
//Insert a vertex
person = graph.addVertex()
person.property("names", "Robert")
person.property("names", "Bob")
graph.tx().commit()
//Now query it
g.V().has("names", "Bob").count().next() //1
g.V().has("names", "Robert").count().next() //1
```