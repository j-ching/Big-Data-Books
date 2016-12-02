什么是OLAP(联机分析处理)？

这个是和数据处理非常相关的一个概念。接触过BI(商务智能)的同学一定清楚。 数据处理大致可以分成两大类：联机事务处理[OLTP](http://baike.baidu.com/view/277075.htm)（on-line transaction processing）、联机分析处理OLAP（On-Line Analytical Processing); OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。**通俗的讲，就是对数据的增删改查等操作**。  OLAP是[数据仓库](http://baike.baidu.com/view/19711.htm)系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。**通俗的讲，就是对数据按不同维度的聚合，维度的上钻，下卷等。**

** **OLAP可以分为ROLAP，MOLAP和HOLAP

ROLAP： 使用关系型数据库或者扩展的关系型数据库来管理数据仓库数据，而OLAP中间件支持其余的功能。ROLAP包含了每个后端关系型数据库的优化，聚合，维度操作逻辑的实现，附件的工具以及服务等。所以ROLAP比MOLAP有更好的可伸缩性。 比较典型的ROLAP有mondrian, Presto(facebook)。 目前阿里的**DRDS**也可以看作是ROLAP的框架

MOLAP： 通过基于数据立方体的多位存储引擎，支持数据的多位视图。即通过将多维视图直接映射到数据立方体上，使用数据立方体能够将预计算的汇总数据快速索引。比较典型的MOLAP框架有kylin(apache), Lylin(ebay)、pinot(linkedin)和**druid**

** 也就是说MOLAP是空间换时间，即把所有的分析情况都物化为物理表或者视图，查询的时候直接从相应的物化表中获取数据， 而ROLAP则通过按维度分库，分表等方式，实现单一维度下的快速查询，通过分布式框架，并行完成分析任务，来实现数据的分析功能。MOLAP 实现较简单，但当分析的维度很多时，数据量呈指数增长，而ROLAP在技术实现上要求更高，但扩展性也较好。**

HOLAP： 混合OLAP结合ROLAP和MOLAP，得益于ROLAP较大的可伸缩性和MOLAP的快速查询。

更多的关于OLAP的知识，推介大家看机械工业出版社出版的《数据挖掘-概念与技术》
