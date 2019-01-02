
![](../img/gremlin-logo.png)

Gremlin是JanusGraph的查询语言，用于从图中检索数据和更新数据。 Gremlin是一种面向路径的语言，它能够简洁地表示复杂的图形遍历和多步操作。 Gremlin是一种函数式语言，遍历运算被链接在一起形成类似路径的表达式。 例如，“从Hercules，遍历他的父亲，然后他父亲的父亲，并返回祖父的名字。”

Gremlin是Apache TinkerPop的一个组件。 它独立于JanusGraph开发，并且支持大多数的图数据库。 通过Gremlin查询语言在JanusGraph基础上开发的应用程序，用户可以避免被数据库绑定，因为他们的应用程序可以迁移到支持Gremlin的其他图数据库。

本节是Gremlin查询语言的简要概述。 有关Gremlin的更多信息，请参阅以下资源：

* Complete Gremlin Manual: Gremlin的参考手册。
* Gremlin Console Tutorial: 学习如何有效地使用Gremlin控制台以交互方式遍历和分析图形。
* Practical Gremlin Book: 图数据库和Gremlin查询语言的入门指南。
* Gremlin Recipes: Gremlin的最佳实践和常见遍历模式的集合。
* Gremlin Language Drivers: 使用不同的编程语言连接到Gremlin服务器，包括Go，JavaScript，.NET / C＃，PHP，Python，Ruby，Scala和TypeScript。
* Gremlin Language Variants: 学习如何在编程语言中嵌入Gremlin。
* Gremlin for SQL developers: 使用SQL查询数据的方式来学习Gremlin。

## 6.1. 遍历介绍

Gremlin查询是一系列从左到右的计算操作/函数。 下面通过第3章“入门”中讨论的Gods图来展示一个简单的祖父查询的示例。

	gremlin> g.V().has('name', hercules').out('father').out('father').values('name')
	==>saturn

上面查询的解读：

1. g：当前图的遍历句柄。
2. V：图中所有的顶点。
3. has(‘name’, ‘hercules’)：过滤出顶点name为hercules的顶点。
4. out(‘father’)：从hercules顶点遍历出边为father的边。
5. out(‘father’)：从hercules的father顶点遍历出边为father的边。
6. name：获取hercules祖父顶点的name属性的值。

总之，这些步骤构成了类似路径的遍历查询。 每个步骤都可以分解并显示其结果。 在构建更大，更复杂的查询时，这种构建遍历/查询的方式很有用。


	==>graphtraversalsource[janusgraph[cql:127.0.0.1], standard]
	gremlin> g.V().has('name', 'hercules')
	==>v[24]
	gremlin> g.V().has('name', 'hercules').out('father')
	==>v[16]
	gremlin> g.V().has('name', 'hercules').out('father').out('father')
	==>v[20]
	gremlin> g.V().has('name', 'hercules').out('father').out('father').values('name')
	==>saturn

对于正确性检查，通常可以查看每个返回值的属性值，而不是查看他们的id。

	gremlin> g.V().has('name', 'hercules').values('name')
	==>hercules
	gremlin> g.V().has('name', 'hercules').out('father').values('name')
	==>jupiter
	gremlin> g.V().has('name', 'hercules').out('father').out('father').values('name')
	==>saturn

注意相关的遍历，展示了Hercules的整个父系树分支。 提供这种更复杂的遍历以展示语言的灵活性和可读性。 对Gremlin的有效掌握为JanusGraph用户提供了快速查询底层图结构遍历的能力。


	gremlin> g.V().has('name', 'hercules').repeat(out('father')).emit().values('name')
	==>jupiter
	==>saturn

下面提供了一些遍历示例。

	gremlin> hercules = g.V().has('name', 'hercules').next()
	==>v[1536]
	gremlin> g.V(hercules).out('father', 'mother').label()
	==>god
	==>human
	gremlin> g.V(hercules).out('battled').label()
	==>monster
	==>monster
	==>monster
	gremlin> g.V(hercules).out('battled').valueMap()
	==>{name=nemean}
	==>{name=hydra}
	==>{name=cerberus}

每一步（由分隔表示）是对上一步计算出的对象进行操作的函数。 Gremlin语言中有许多步（参见Gremlin Steps）。 通过简单地改变步骤或着改变步骤的顺序，可以实现不同的遍历。 下面的例子返回所有与Hercules战斗相同怪物的人的名字，并且除去Hercules本身（即“共同战士”或者“盟友”）。

鉴于神的图形只有一个战斗者（Hercules），另一个战斗者（为了举例）被添加到图中，Gremlin展示了如何将顶点和边添加到图形中。


	gremlin> theseus = graph.addVertex('human')
	==>v[3328]
	gremlin> theseus.property('name', 'theseus')
	==>null
	gremlin> cerberus = g.V().has('name', 'cerberus').next()
	==>v[2816]
	gremlin> battle = theseus.addEdge('battled', cerberus, 'time', 22)
	==>e[7eo-2kg-iz9-268][3328-battled->2816]
	gremlin> battle.values('time')
	==>22

添加顶点时，可以选择是否指定顶点标签。 但是添加边时必须指定边标签。 可以在顶点和边上设置作为键值对的属性。 使用SET或LIST基数定义的属性键，必须使用addProperty向顶点添加此属性。


	gremlin> g.V(hercules).as('h').out('battled').in('battled').where(neq('h')).values('name')
	==>theseus

上面的例子有4个链接函数：out，in，except和values（即name是值的简写（’name’））。 每个的函数在下面逐条列出，其中V是顶点而U是任何对象，其中V是U的子集。

1. out: V -> V
2. in: V -> V
3. except: U -> U
4. values: V -> U

将函数链接在一起时，传入类型必须与传出类型匹配，其中U匹配任何内容。 因此，上面的“共同战斗/盟友”遍历是正确的。

> 注意：本节中介绍的Gremlin概述重点介绍了在Gremlin控制台中Gremlin-Groovy语言实现版本的使用。 Gremlin的其他语言驱动和实现也是可以使用的。

## 6.2. 遍历迭代

Gremlin控制台其中的一个特性是它从gremlin>prompt自动迭代所有的查询结果。 这在REPL环境中很好用，而且它将结果作为String类型来展示。 当你开始编写Gremlin应用程序时，了解如何显式迭代遍历非常重要，因为应用程序的遍历不会自动迭代。 以下是迭代遍历的一些常用方法：

* iterate() - 预期或者可以忽略空值。
* next() - 获取一个结果，一定要先通过hasNext()判断。
* next(int n) - 获取第n个结果，一定要先通过hasNext()判断。
* toList() - 获取所有的结果作为一个list，如果没有结果则返回空列表。

下面使用Java代码示例来演示这些概念：

	Traversal t = g.V().has("name", "pluto"); // Define a traversal
	// Note the traversal is not executed/iterated yet
	if (t.hasNext()) { // Check if results are available
	    pluto = g.V().has("name", "pluto").next(); // Get one result
	    g.V(pluto).drop().iterate(); // Execute a traversal to drop pluto from graph
	}
	// Note the traversal can be cloned for reuse
	Traversal tt = t.asAdmin().clone();
	if (tt.hasNext()) {
	    System.err.println("pluto was not dropped!");
	}
	List<Vertex> gods = g.V().hasLabel("god").toList(); // Find all the gods