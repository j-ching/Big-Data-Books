[TOC]

## chukwa
chukwa 是apache 开源的针对大规模分布式系统Log 收集和分析的项目， 建立在hadoop的基础上。 架构图如下

![架构图](http://www.goingio.com/wordpress/wp-content/uploads/2016/08/chukwa_architecture.png)

chukwa可以分为五个部分：
+ hadoop和hbase集群，用于chukwa处理数据
+ 一个或多个agent进程， 将监控的数据发送到hbase集群。 启动有agent进程的节点被称作监控源节点
+ Solr云集群，用于存储索引数据文件
+ 数据分析脚本， 概括hadoop集群健康度
+ HICC， chukwa 的可视化工具

### chukwa agent
chukwa agent 进程运行在每一台需要被监控的机器上， 它的职责就是收集服务器上所有的数据， 收集的方式可以是运行一个unix命令，tail 一个文件， 或者监控入口的udp包。
每个特定的数据源都对应着一个单独的adaptor。 adaptor是运行在agent进行内部的可以单独被加载的模块。 这也就意味着，每一种数据源对应一个独立的adaptor， 比如给一个监控的文件添加一个adaptor或者给一个正在被执行的unix添加一个adaptor。 每个adaptor都有自己唯一的命名， 如果没有特别指明， 将会根据adaptor的类型和参数自动生成一个名称。
目前chukwa包含了很多adaptor，你也可以自己开发adaptor。

### chukwa Pipeline
chukwa Pipeline 的职责是接收从agent 过来的数据， 提取，输入到指定的存储中。 大多数情况下， Pipeline 会将数据写入到hbase或hdfs。

#### HBase

```
	<property>
	  <name>chukwa.pipeline</name>
	  <value>org.apache.hadoop.chukwa.datacollection.writer.hbase.HBaseWriter</value>
	</property>

```

#### HDFS

```
	<property>
	  <name>chukwa.pipeline</name>
	  <value>org.apache.hadoop.chukwa.datacollection.writer.parquet.ChukwaParquetWriter</value>
	</property>

```
上面的配置，我们通过配置Pipeline，直接将一系列的数据写入到hbase，或者将一系列的文件直接写入到hdfs，但是我们也可以改变pipe class. 一个最有用的Pipeline class 是 pipelinestagewriter, 它可以让我们在收集的数据流前或者后，配置一系统的PipelineableWriters。 例如 SocketTeeWriter class 将数据通过chukwa agent的Socket送到其他程序中。

```
	<property>
	  <name>chukwa.pipeline</name>
	  <value>org.apache.hadoop.chukwa.datacollection.writer.SocketTeeWriter,org.apache.hadoop.chukwa.datacollection.writer.parquet.ChukwaParquetWriter</value>
	</property>
```


[chukwa](http://chukwa.apache.org/)


