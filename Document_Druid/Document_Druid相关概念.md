[TOC]

**Data**

druid的数据格式和关系型数据库数据较为类似， 如下：
`timestamp publisher advertiser gender country click price  
2011-01-01T01:01:35Z [bieberfever.com](http://bieberfever.com) [google.com](http://google.com) Male USA 0 0.65  
2011-01-01T01:03:63Z [bieberfever.com](http://bieberfever.com) [google.com](http://google.com) Male USA 0 0.62  
2011-01-01T01:04:51Z [bieberfever.com](http://bieberfever.com) [google.com](http://google.com) Male USA 1 0.45  
2011-01-01T01:00:00Z [ultratrimfast.com](http://ultratrimfast.com) [google.com](http://google.com) Female UK 0 0.87  
2011-01-01T02:00:00Z [ultratrimfast.com](http://ultratrimfast.com) [google.com](http://google.com) Female UK 0 0.99  
2011-01-01T02:00:00Z [ultratrimfast.com](http://ultratrimfast.com) [google.com](http://google.com) Female UK 1 1.53`


熟悉OLAP的同学，对以下这些概念一定不陌生，druid也把数据分为以下三个部分：

Timestamp Column：将时间单独处理，是因为druid所有的操作都是围绕时间轴来进行的。

Dimension Columns：维度字段，是数据的属性， 一般被用来过滤数据。上面的例子，我们有四个维度, publisher, advertiser, gender, country. 他们每一个都可以看是数据立方体的一个轴，都可以用来用来做横切。

Metric Columns: 度量字段，是用来做聚合或者相关计算的。 上边的数据， click和price是俩个度量。度量是可以衡量的数据，一般可以有如下的操作，count ，sum等等

  


**ROLL-UP**

  


roll-up （上卷）是olap的基本操作(除此之外还有下钻，切片等， 基本理论是一样的)。 在数据统计里，由于数据量太多，一般对细分的数据不是特别干兴趣，或者说没有太大关注的意义。

但是按照维度的汇总或者统计，确实很有用的。druid通过一个roll-up的处理，将原始数据在注入的时候就进行汇总处理。roll-up 是在维度过滤之前的第一层聚合操作，如下：

`GROUP BY timestamp, publisher, advertiser, gender, country  
:: impressions = COUNT(1), clicks = SUM(click), revenue = SUM(price)`

聚合后数据就变成了如下的样子

`timestamp publisher advertiser gender country impressions clicks revenue  
2011-01-01T01:00:00Z [ultratrimfast.com](http://ultratrimfast.com) [google.com](http://google.com) Male USA 1800 25 15.70  
2011-01-01T01:00:00Z [bieberfever.com](http://bieberfever.com) [google.com](http://google.com) Male USA 2912 42 29.18  
2011-01-01T02:00:00Z [ultratrimfast.com](http://ultratrimfast.com) [google.com](http://google.com) Male UK 1953 17 17.31  
2011-01-01T02:00:00Z [bieberfever.com](http://bieberfever.com) [google.com](http://google.com) Male UK 3194 170 34.01`


我们可以看到，roll-up可以压缩我们需要保存的数据量。 druid 通过roll-up 减少了 我们存储在后台的数据量。 但这种缩减是有损失的， 当我们做了roll-up, 我们就无法查询细分的数据了。 或许，我们在可以在rollup的时候，将其粒度控制在我们可以查询到我们需要查看的最细数据为止。druid可以通过 queryGranularity 来控制注入数据的粒度。 最小的queryGranularity 是 millisecond(毫秒级)

Sharding the Data

druid的数据分片称为 segments, 一般的，druid会通过时间来进行分片， 上面例子中我们聚合后的数据，可以按小时分为俩片，如下

Segment `sampleData_2011-01-01T01:00:00:00Z_2011-01-01T02:00:00:00Z_v1_0` contains

    2011-01-01T01:00:00Z  [ultratrimfast.com](http://ultratrimfast.com)  [google.com](http://google.com)  Male   USA     1800        25     15.70
     2011-01-01T01:00:00Z  [bieberfever.com](http://bieberfever.com)    [google.com](http://google.com)  Male   USA     2912        42     29.18


Segment `sampleData_2011-01-01T02:00:00:00Z_2011-01-01T03:00:00:00Z_v1_0` contains

    2011-01-01T02:00:00Z [ultratrimfast.com](http://ultratrimfast.com) [google.com](http://google.com) Male UK 1953 17 17.31
    2011-01-01T02:00:00Z  [bieberfever.com](http://bieberfever.com) [google.com](http://google.com) Male UK 3194 170 34.01


segment 是个包含数据的独立的容器， 内部的数据以时间分割。 segment 为聚合的列做索引，数据依赖索引，按列方式存储。 所以druid得查询就转为了如何扫描segments了。

segment 由datasource， interval, version, 和一个可选的分区号唯一的确定。 例如上面例子中，我们的segment的名字就是这种格式dataSource_interval_version_partitionNumber

**写到这里，大家应该也有了初步的了解，druid 在注入的数据的时候，就已经将索引按照指定的格式处理好，并保存在deepstore中， 其余的查询都转换为了对数据的扫描过程。 所以druid是典型的MOLAP**

Indexing the Data

druid能达到这样的速度，主要取决于数据的存储格式。 druid为数据创建了不可变的数据镜像， 并已便于分析搜索的的结构存储下来。 druid是列存储的， 也就是说，每一个单独的列都是分开存储的。查询过程中，也只有与查询有关联的列参与。 druid对于只有扫描的查询更有优势。 不同的列可以调用不同的压缩方式。不同的列也可以有不同的索引。

**druid的索引是segment级别的。**

Loading the Data

druid有俩种数据load的方式，一种是realtime的，一种是batch的。 druid的实时数据注入是很费力的。 Exactly once semantics are not guaranteed with real-time ingestion in Druid, although we have it on our roadmap to support this. Batch ingestion provides exactly once guarantees and segments created via batch processing will accurately reflect the ingested data。常用的做法是通过real-time 方式来管理实时的数据分析，通过batch 方式来管理精确备份的数据。

Querying the Data

druid原生的查询是以json参数调用http接口，社区也分享了各种其他语言的查询库， 包括sql

druid 主要是为单表的数据操作儿设计的，所以目前不支持join操作。 很多产品需要在etl阶段做join， 可以把join放在数据注入druid之前来进行。

The Druid Cluster


druid集群是由很多功能不同的节点组成的。

  


**Historical Nodes：historical nodes 可以看做是druid集群的脊椎， 它将segment固话到本地，供集群查询时使用。 historical nodes 采用了一个无共享架构设计， 它知道如何去加载segment, 删除segment以及如果基于segment查询。**

**Broker Nodes：broker Nodes 是客户端和相关应用从druid集群上查询数据的节点，它的职责是对客户端过来的查询做负载，聚集和合并查询结果。 broker节点知道每个segment都在哪儿**

**Coordinator Nodes：coordinator nodes 用来管理druid集群放在historical nodes上的segment。coordinatenodes 告诉historical nodes去加载新的segment， 移除就得segment， 对节点上的segment做均衡**

**Real-time Processing：实时数据处理可以在单点实时节点或者索引服务(indexing service)完成， 实时的逻辑在这二者上是很常见的。实时处理主要包括加载数据，创建索引(创建segment)， 以及将segment迁移到historical nodes。经过实时处理后的数据及可查询。迁移处理也是无损的， 迁移后数据仍然是可以查询的。**

**overload Nodes： **


External Dependencies

druid集群需要依赖：

**Zookeeper 用于集群内部通讯**

**Metadata Storage** 用户存储segment，configuration 等的metadata信息； 服务创建segments后，会向metadatastore中写一个新的标记， coordinatenode监控metadatastore来获取有哪些新的数据需要被重新load，或者有哪些旧的数据需要被去除。查询的时候并不需 要metadatastor的数据。 在生产集群中，mysql 和postgresql是比较常用的metadatastor， derby可以用于单机测试环境

**Deep Storage** deepstorage作为segments一种持久的备份。 服务创建segments后，上传到deepstore。 coordinatenode从deepstorage下载segments。查询的时候也不会用到deepstorage。 常用的deepstorage有S3和hdfs。


