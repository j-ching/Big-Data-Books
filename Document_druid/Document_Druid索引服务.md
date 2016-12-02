对于索引服务的配置，可以查看[索引服务]

  


索引服务是一个高性能呢，分布式服务，它用来运行相关联的索引任务。 索引服务可以创建或者删除druid的segments。索引服务有master/slave的架构。

索引服务由如下几个部分组成：

  1. 一个可以运行单独任务的peon component
  2. 一个middle manager 用于管理peon
  3. 一个overload 用于将任务分发到middle minager

overloads 和 middle managers 可以用于在相同的节点上或者多个节点上， 这时middle managers和penons 运行在相同的节点上

  


Indexing Service Overview

  