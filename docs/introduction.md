# Part I. 简介

For full documentation visit [mkdocs.org](https://mkdocs.org).

## 1. JanusGraph有点
对对对

### 1.1. JanusGraph的主要优点
对对对
### 1.2. 用Apache Cassandra作为JanusGraph的优点
对对对
### 1.3. 用Hbase作为JanusGraph的优点
对对对
### 1.4. JanusGraph和CAP定理
对对对
## 2. 架构概览

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.

## 3. 快速开始

图3.1 神的图关系
![graph-of-the-gods-2](./img/graph-of-the-gods-2.png)
*Above: Cupcake indexer in progress*

### 3.1. 下载JanusGraph和运行Gremlin控制台
JanusGraph可以从项目仓库中下载一个发布的<a href="https://github.com/JanusGraph/janusgraph/releases" target="_blank">版本</a> 一旦下载并解压，一个Gremlin的控制台就可以打开。Gremlin控制台是与JanusGraph一起分发的[REPL](http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)（即交互式shell），并且仅与标准Gremlin控制台不同，因为JanusGraph是预先安装和预加载的包。或者，用户可以通过从中央存储库下载JanusGraph包来选择在现有的Gremlin控制台中安装和激活JanusGraph。但是，在下面的示例中，使用了 janusgraph.zip，请确保解压缩了下载的zip文件。

>*重要: JanusGraph依赖Java8（标准版）。建议使用Oracle Java 8。JanusGraph的shell脚本希望$JAVA_HOME环境变量指向安装JRE或JDK的目录。*



		$ unzip janusgraph-0.3.1-hadoop2.zip
		Archive:  janusgraph-0.3.1-hadoop2.zip
		creating: janusgraph-0.3.1-hadoop2/
		...
		$ cd janusgraph-0.3.1-hadoop2
		$ bin/gremlin.sh

         	\,,,/
         	(o o)
		-----oOOo-(3)-oOOo-----
		09:12:24 INFO  org.apache.tinkerpop.gremlin.hadoop.structure.HadoopGraph  - HADOOP_GREMLIN_LIBS is set to: /usr/local/janusgraph/lib
		plugin activated: tinkerpop.hadoop
		plugin activated: janusgraph.imports
		gremlin>

Gremlin控制台使用[Apache Groovy](http://www.groovy-lang.org/)解释命令。Groovy是Java的超集，它具有各种简化标记，使交互编程更容易。同样地，Gremlin-Groovy是Groovy的超集，具有各种速记符号，使图形遍历变得容易。下面的基本示例演示处理数字、字符串和映射。本教程的其余部分将讨论特定于图形的构造。

		gremlin> 100-10
		==>90
		gremlin> "JanusGraph:" + " The Rise of Big Graph Data"
		==>JanusGraph: The Rise of Big Graph Data
		gremlin> [name:'aurelius', vocation:['philosopher', 'emperor']]
		==>name=aurelius
		==>vocation=[philosopher, emperor]

> 提示

> 关于[Apache Tinkerpop](http://tinkerpop.apache.org/docs/3.3.3/reference),[SQLEGremlin](http://sql2gremlin.com/)和[Gremlin Recipes](http://tinkerpop.apache.org/docs/3.3.3/recipes/)的更多Gremlin的使用信息。

### 3.2. 将Gods的图关系导入到JanusGraph

下面的示例将打开一个JanusGraph图形实例，并加载上面所示的Gods数据集的图。JanusGraphFactory提供了一组静态打开方法，每个方法都采用配置作为参数，并返回一个图形实例。本教程在使用BerkeleyDB存储后端和Elasticsearch索引后端的配置上调用这些开放方法，然后使用帮助类GraphOfTheGodsFactory加载Gods的图。本节跳过配置细节，但是关于存储后端、索引后端及其配置的附加信息可在Part III,“存储后端”、Part IV“索引后端”和第15章“配置参考”中获得。

	gremlin> graph = JanusGraphFactory.open('conf/janusgraph-berkeleyje-es.properties')
	==>standardjanusgraph[berkeleyje:../db/berkeley]
	gremlin> GraphOfTheGodsFactory.load(graph)
	==>null
	gremlin> g = graph.traversal()
	==>graphtraversalsource[standardjanusgraph[berkeleyje:../db/berkeley], standard]


在返回新构造的图之前，JanusGraphFactory.open()和GraphOfTheGodsFactory.load()方法执行以下操作：

 1. 在图上创建全局和顶点中心索引集合。
 2. 将所有顶点添加到图形以及它们的属性。
 3. 将所有的边加上它们的属性。

详情请参阅[GraphOfTheGodsFactory source code](https://github.com/JanusGraph/janusgraph/blob/master/janusgraph-core/src/main/java/org/janusgraph/example/GraphOfTheGodsFactory.java)源代码。

对于那些使用JanusGraph/Cassandra（或JanusGraph/HBase）的用户，请确保使用conf/janus.-cql-es.properties（或conf/janus.-hbase-es.properties）和GraphOfTheGodsFactory.load()。

	gremlin> graph = JanusGraphFactory.open('conf/janusgraph-cql-es.properties')
	==>standardjanusgraph[cql:[127.0.0.1]]
	gremlin> GraphOfTheGodsFactory.load(graph)
	==>null
	gremlin> g = graph.traversal()
	==>graphtraversalsource[standardjanusgraph[cql:[127.0.0.1]], standard]

### 3.3. 全局图索引
访问图形数据库中数据的典型模式是首先使用图形索引定位到图形中的入口点。该入口点是元素（或一组元素）-即顶点或边。从条目元素中，Gremlin路径描述描述了如何通过显式图结构遍历到图中的其他元素。

给定name属性上有一个唯一的索引，可以检索Saturn顶点。然后可以检查属性映射（即Saturn的密钥/值对）。正如所展示的，土星顶点的名字是“Saturn”，年龄为10000岁，是一种“titan”。Saturn的孙子可以通过一个遍历表述：“谁是Saturn的孙子？”“父亲”的倒数是“孩子”。结果就是Hercules。

	gremlin> saturn = g.V().has('name', 'saturn').next()
	==>v[256]
	gremlin> g.V(saturn).valueMap()
	==>[name:[saturn], age:[10000]]
	gremlin> g.V(saturn).in('father').in('father').values('name')
	==>hercules

属性位置也在图形索引中。属性位置是一个边属性。因此，JanusGraph可以索引图索引中的边。可以查询在雅典50公里内发生的所有事件（纬度：37.97和长度：23.72）。然后，给定这些信息，哪些顶点参与了这些事件。

	gremlin> g.E().has('place', geoWithin(Geoshape.circle(37.97, 23.72, 50)))
	==>e[a9x-co8-9hx-39s][16424-battled->4240]
	==>e[9vp-co8-9hx-9ns][16424-battled->12520]
	gremlin> g.E().has('place', geoWithin(Geoshape.circle(37.97, 23.72, 50))).as('source').inV().as('god2').select('source').outV().as('god1').select('god1', 'god2').by('name')
	==>[god1:hercules, god2:hydra]
	==>[god1:hercules, god2:nemean]

图索引是JanusGraph中的一种索引结构。JanusGraph自动选择图形索引，以回答要求满足一个或多个约束（例如有或间隔）的所有顶点（g.V）或所有边（g.E）的问题。anusGraph中索引的第二个方面被称为顶点中心索引。顶点中心指数用于加速图中的遍历。稍后将描述顶点中心指数。

#### 3.3.1 图遍历实例
> Hercules，Jupiter和Alcmene的儿子，“蕴藏着超人的力量”。Hercules是个Demigod(半神)，因为他的父亲是God，他的母亲是人类。Juno，Jupiter的妻子，对Jupiter的不忠感到愤怒。为了报仇，她用暂时的精神错乱蒙蔽了Hercules，并导致他杀死他的妻子和孩子。为了赎罪， Oracle of Delphi的神谕命令Hercules为Eurystheus服务。Eurystheus指派给Hercules为12名劳工。

在上一节中，它证明了Saturn的孙子是Hercules。这可以用一个循环来表示。本质上，Hercules是沿着“父亲”路径离开Hercules2步的顶点。

	gremlin> hercules = g.V(saturn).repeat(__.in('father')).times(2).next()
	==>v[1536]


Hercules是demigod(半神半人)。为了证明Hercules是半人半神，他父母的起源必须加以考证。有可能从Hercules顶点到他的父母。最后，可以确定它们分别产生“神”和“人”的类型。

	 gremlin> g.V(hercules).out('father', 'mother')
	 ==>v[1024]
	 ==>v[1792]
	 gremlin> g.V(hercules).out('father', 'mother').values('name')
	 ==>jupiter
	 ==>alcmene
	 gremlin> g.V(hercules).out('father', 'mother').label()
	 ==>god
	 ==>human
	 gremlin> hercules.label()
	 ==>demigod

迄今为止的例子是关于罗马万神殿中各种角色的基因谱系。[属性图模型](http://tinkerpop.apache.org/docs/3.3.3/reference#intro)具有足够的表达能力，可以表示多种类型的事物和关系。通过这种方式，《神图》还确定了Hercules的各种英勇事迹——他著名的12项劳动。在上一节中，人们发现Hercules参与了雅典附近的两场战役。通过穿越Hercules顶点之外的战斗边缘，有可能探索这些事件。


    gremlin> g.V(hercules).out('battled')
    ==>v[2304]
    ==>v[2560]
    ==>v[2816]
    gremlin> g.V(hercules).out('battled').valueMap()
    ==>[name:[nemean]]
    ==>[name:[hydra]]
    ==>[name:[cerberus]]
    gremlin> g.V(hercules).outE('battled').has('time', gt(1)).inV().values('name')
    ==>cerberus
    ==>hydra

战斗边的属性time由一个顶点的vertex-centric指数索引。根据约束/滤波器按时检索入射到Hercules的战斗边缘比进行所有边缘的线性扫描和滤波(通常为O(log n)，其中n是入射边的数目)要快。JanusGraph足够智能，可以在使用时使用顶点中心索引。Gremlin表达式的toString()显示分解为各个步骤。

    gremlin> g.V(hercules).outE('battled').has('time', gt(1)).inV().values('name').toString()
    ==>[GraphStep([v[24744]],vertex), VertexStep(OUT,[battled],edge), HasStep([time.gt(1)]), EdgeVertexStep

#### 3.3.2 更复杂的图遍历示例

冥府深处的冥王星。他与Hercules的关系因Hercules与他的宠物Cerberus搏斗而紧张。然而，Hercules是他的侄子，他应该如何让Hercules为他的傲慢付出代价？
下面的Gremlin遍历在Gods的图上提供更多的例子。每个遍历的解释在先前行中提供为a//注释。

##### 3.3.2.1 Tartarus的同居者

    gremlin> pluto = g.V().has('name', 'pluto').next()
    ==>v[2048]
    gremlin> // who are pluto's cohabitants?
    gremlin> g.V(pluto).out('lives').in('lives').values('name')
    ==>pluto
    ==>cerberus
    gremlin> // pluto can't be his own cohabitant
    gremlin> g.V(pluto).out('lives').in('lives').where(is(neq(pluto))).values('name')
    ==>cerberus
    gremlin> g.V(pluto).as('x').out('lives').in('lives').where(neq('x')).values('name')
    ==>cerberus

##### 3.3.2.2. Pluto’s 兄弟

    gremlin> // where do pluto's brothers live?
    gremlin> g.V(pluto).out('brother').out('lives').values('name')
    ==>sky
    ==>sea
    gremlin> // which brother lives in which place?
    gremlin> g.V(pluto).out('brother').as('god').out('lives').as('place').select('god', 'place')
    ==>[god:v[1024], place:v[512]]
    ==>[god:v[1280], place:v[768]]
    gremlin> // what is the name of the brother and the name of the place?
    gremlin> g.V(pluto).out('brother').as('god').out('lives').as('place').select('god', 'place').by('name')
    ==>[god:jupiter, place:sky]
    ==>[god:neptune, place:sea]



最后，冥王星(pluto)生活在Tartarus，因为他不关心生死。另一方面，他的兄弟们基于对那些地方某些品质的爱来选择他们的位置

    gremlin> g.V(pluto).outE('lives').values('reason')
    ==>no fear of death
    gremlin> g.E().has('reason', textContains('loves'))
    ==>e[6xs-sg-m51-e8][1024-lives->512]
    ==>e[70g-zk-m51-lc][1280-lives->768]
    gremlin> g.E().has('reason', textContains('loves')).as('source').values('reason').as('reason').select('source').outV().values('name').as('god').select('source').inV().values('name').as('thing').select('god', 'reason', 'thing')
    ==>[god:neptune, reason:loves waves, thing:sea]
    ==>[god:jupiter, reason:loves fresh breezes, thing:sky]

本节概述JanusGraph的架构和优点，然后使用小教程数据集快速浏览JanusGraph特性。
