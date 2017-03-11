[TOC]

Druid 是目前比较流行的高性能的，分布式列存储的OLAP框架(具体来说是MOLAP)。它有如下几个特点：

一. 亚秒级查询

druid提供了快速的聚合能力以及亚秒级的OLAP查询能力，多租户的设计，是面向用户分析应用的理想方式。

二.实时数据注入

druid支持流数据的注入，并提供了数据的事件驱动，保证在实时和离线环境下事件的实效性和统一性

三.可扩展的PB级存储

druid集群可以很方便的扩容到PB的数据量，每秒百万级别的数据注入。即便在加大数据规模的情况下，也能保证时其效性

四.多环境部署

druid既可以运行在商业的硬件上，也可以运行在云上。它可以从多种数据系统中注入数据，包括hadoop，spark，kafka，storm和samza等

五.丰富的社区

druid拥有丰富的社区，供大家学习。

** 关于Druid**

Druid is an open-source analytics data store designed for business intelligence ([OLAP](http://en.wikipedia.org/wiki/Online_analytical_processing)) queries on event data. Druid provides low latency (real-time) data ingestion, flexible data exploration, and fast data aggregation. Existing Druid deployments have scaled to trillions of events and petabytes of data. Druid is most commonly used to power user-facing analytic applications.

## Key Features

**Sub-second OLAP Queries** Druid’s column orientation and inverted indexes enable complex multi-dimensional filtering and scanning exactly what is needed for a query. Aggregate and filter on data in milliseconds.

**Real-time Streaming Ingestion** Typical analytics databases ingest data via batches. Ingesting an event at a time is often accompanied with transactional locks and other overhead that slows down the ingestion rate. Druid employs lock-free ingestion of append-heavy data sets to allow for simultaneous ingestion and querying of 10,000+ events per second per node. Simply put, the latency between when an event happens and when it is visible is limited only by how quickly the event can be delivered to Druid.

**Power Analytic Applications** Druid has numerous features built in for multi-tenancy. Power user-facing analytic applications designed to be used by thousands of concurrent users.

**Cost Effective** Druid is extremely cost effective at scale and has numerous features built in for cost reduction. Trade off cost and performance with simple configuration knobs.

**Highly Available** Druid is used to back SaaS implementations that need to be up all the time. Druid supports rolling updates so your data is still available and queryable during software updates. Scale up or down without data loss.

**Scalable** Existing Druid deployments handle trillions of events, petabytes of data, and thousands of queries every second.
  

