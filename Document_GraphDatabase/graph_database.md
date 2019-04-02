# 图数据库
图(Graph)是一种数据模型， 它包括节点，边以及他们附带的一系列属性的集合。 点其实就是大家所熟知的实体，而边标识了任何俩个节点之间存在的关系。 属性则标识了点或者边所具有的一系列特征。

图数据库(Graph Database)，是使用图的结构来表现和存储具有图语义的数据，并快速的进行查询。 图数据的关键在于存储的是图的数据，图的数据直接将节点与节点之间的关系也保存下来，这样使得在查询的时候，俩个检点的数据可以进行直接的关联，有的甚至只需要关联一步就可以获取。图数据库将数据之间的关系最为重中之重优先进行考虑。所以使用图数据库查询关系数据很快。图数据库能够直观的展示数据之间的关系，对于高度互联的数据非常的有用。

![](media/15541134360669.jpg)


# 图模型
图模型有俩种， 一种是Labeled-property graph,  一种是Resource Description Framework(RDF)

## Labeled-property graph 
在属性标签图模型中， 图是由一系列节点， 关系，属性和标签构成的。 节点和他们之间的关系都会被命名，使用KV的方式来存储其属性。节点通过label来分组。而边有俩个属性，一个是开始节点和结束节点，另外一个是方向， 这样就构成了一个有向图。关系节点也可以有属性， 这样就为节点的关系提供了附加的信息和语义。保存关系的方向有助于图的快速遍历。


## Resource Description Framework(RDF)
在资源描述框架模型中， 附加的属性都会被标识为一个独立的节点。 想象一个场景， 当需要添加一个name属性在图里person的节点上， 在属性标签图模型中， 只需要在person的节点上添加一个name属性就可以了。 而在RDF模型中，需要添加一个单独的hasName节点关联到原来的person节点上。 准确的来讲， RDF图模型，由节点和弧组成。 一个RDF的图表示为： 一个主题节点，一个对象节点， 和一个谓词的弧。节点可以留空， and/or 可以使用URIref来标识。弧也可以通过URIref标识。弧有俩种类型， 普通文字和类别文字。 普通文字具有词法形式和可选的语言标签。类型化文字由具有标识特定数据类型的URIref的字符串组成。 当数据没有URI的时候，可以使用空白节点来说明数据的状态。 

# 图类型

+ **Social Graph** 关于人的连接
+ **Intent Graph** 关于动机和意图的连接
+ **Consumption graph** 也被称做 "payment graph"， 消费图大量使用在零售行业
+ **Interest graph** 关于一个人的兴趣， 经常在社交网络中使用
+ **Mobile graph** 

# 图数据库性能
图数据库的查询最后总会被定位到图的一部分，其余的不相关的数据不会被搜索， 所以在大数据的实时分析查询中比较有优势。所以图数据库的性能与需要遍历的数据的大小有关， 而整体数据存储量的大小，对查询的数据影响不大。


# 图数据库属性
图数据库是图查询的一个有力的工具。 如计算俩个节点之间的最短路径。 使用图数据库， 可以很自然的使用图的方式来执行图的查询。


## 底层存储引擎
图数据库底层的存储机制是可变的。有一些图数据库底层依赖关系型引擎， 将图数据存储成table。还有一些使用KV存储或者基于文档的数据库来存储。 其中一个最重要的数据库是ArangoDB。  ArangoDB是一个多模型数据库， 图模型是它众多模型中的一种。 它通过将边和节点保存在单独的文档集合中来存储图形。节点可以存储为任意的文档， 但是连接俩个不同节点的边需要包含俩个特殊的属性，_from和_to.

## 处理引擎

**index-free adjacency** 无索引连接
数据处理性能取决于从一个节点到另外一个节点的访问速度。因为无索引连接强制节点保存直接物理RAM地址， 并物理的指向相邻节点的地址， 这样就实现了一个快速的遍历。具有无索引连接的原生图数据库，并不需要到其他类型的数据结构中去查找节点直接的关联， 一旦一个节点被检索到， 图中的直接关系节点就会被存储在缓存中，这样使得查询比第一次检索更加快速。 然后这种优化也是要付出代价的， 无索引邻接牺牲了不使用图遍历的查询的效率。 


# 比较常见的图数据库

+ AllegroGraph
+ Amazon Neptune
+ AnzoGraph
+ ArangoDB
+ DataStax Enterprise Graph
+ InfiniteGraph
+ JanusGraph
+ MarkLogic
+ Microsoft SQL Server 2017
+ [Neo4j](./graph_database_neo4j.md)
+ OpenLink Virtuoso
+ Oracle Spatial and Graph; part of Oracle Database
+ OrientDB
+ SAP HANA
+ Sparksee
+ Sqrrl Enterprise
+ Teradata Aster

# API以及图查询语言

+ **AQL (ArangoDB Query Language)** – an SQL-like query language used in ArangoDB for both documents and graphs
+ **Cypher Query Language (Cypher)** – a graph query declarative language for Neo4j that enables ad hoc and programmatic (SQL-like) access to the graph.[39]
+ **GraphQL** – Facebook query language for any backend service
+ **Gremlin** – a graph programming language that works over various graph database systems; part of Apache TinkerPop open-source project[40]
+ **SPARQL** – a query language for RDF databases, can retrieve and manipulate data stored in Resource Description Framework format


