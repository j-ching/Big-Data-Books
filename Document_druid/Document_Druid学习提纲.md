

  1. 数据

时间 维度 度量

2\. 毫秒级聚合能力

  3. 数据分片 
    1. 按小时切片
    2. 切片内包含有该时间段内的数据。 既有压缩后的原始数据，又有索引数据。
    3. 查询的关键在于如何找到分片
  4. 分片可以有数据源，时间段，版本和一个可选的分区号来因为确定
  5. 索引数据 
    1. 构造结构化的数据镜像，优化分析查询
    2. 列存储
  6. 加载数据 
    1. 实时 
      1. push： kafka storm sparkstreaming等
      2. pull
    2. 批量 
      1. files - hdfs s3 localfiles 等文件系统
  7. 查询数据 
    1. http接口
    2. sdk
  8. 数据可视化 
    1. pivot
    2. caravel
    3. metabase
  9. 集群 
    1. historical nodes
    2. broker nodes
    3. coordinator nodes
    4. real-time processing 

另外还有(overload node, middle manager)

  10. 相关依赖  

    1. zookeeper
    2. metadata storage
    3. deep storage
  11. 加载的数据格式 
    1. json
    2. csv
    3. tsv
  12. 加载数据规格

{

“dataSchema”: {},

“ioConfig”: {} ,

“tuningConfig”: {}

}

  13. 集群基础环境 
    1. 需要有metadata storage
    2. 需要有zk集群
    3. 需要有deep storage
  14. 集群搭建 
    1. 每种节点不需要自己独立的服务器
    2. overload和coordinator节点需要在同一台服务器，做协同处理
    3. hdfs 配置文件放入_common下
    4. 在与hadoop对接的时候，需要将集群的时间统一(统一为UTC、CTS等)
  15. reduce GC overload the limited

调整mapreduce 内存参数

  16. rebuild index