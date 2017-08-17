# HDFS High Availability Using the Quorum Journal Manager

+ **JournalNodes** standyby 节点与active 节点同步状态，需要俩种节点与journalNode通信. 当active 节点执行了一些namespace的变更操作时， 它会在jns的主要节点上持久的记录一些表更日志, standby 节点，会从jns 中读取这些编辑操作日志， 并持久的监控edit log的变更， 当standby读到这些变更的时候，它会将这些表更运行到自己的namespac上。当发生failover的时候，standby节点要确保自己从journalnode中读取到升级为activenode之前所有的变更，这也保证了namespace的状态与failover之前是同步的。

为了提供快速的failover， standby节点必须不断的更新集群中关于block的位置信息， 为了达到这个目标， Datanode上配置了所有namenode的位置信息， 并且向所有的namenode发送位置和心跳信息




