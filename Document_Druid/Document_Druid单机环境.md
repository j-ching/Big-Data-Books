[TOC]

druid 可以运行在单机环境下，也可以运行在集群环境下。简单起见，我们先从单机环境着手学习。

# **环境要求**

  * java7 或者更高版本
  * linux， macOS或者其他unix系统(不支持windows系统)
  * 8G内存
  * 2核CPU

# 开始

下载并安装druid

    curl -O [http://static.druid.io/artifacts/releases/druid-0.9.1.1-bin.tar.gz](http://static.druid.io/artifacts/releases/druid-0.9.1.1-bin.tar.gz)
    tar -xzf druid-0.9.1.1-bin.tar.gz
    cd druid-0.9.1.1
    

文件夹中有如下几个目录

  * LICENSE 许可证
  * bin/ 可执行脚本
  * conf/* 在集群环境下的配置文件
  * conf-quickstart/* quickstart的配置文件
  * extensions/* druid所有的扩展文件
  * hadoop-dependencies/* druid的hadoop扩展文件
  * lib/* druid 依赖的核心软件包
  * quickstart/* quickstart的数据文件

# ZK安装
druid的分布式协同需要依赖zookeeper，所以我们需要安装zk

    curl [http://www.gtlib.gatech.edu/pub/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz](http://www.gtlib.gatech.edu/pub/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz) -o zookeeper-3.4.6.tar.gz
    tar -xzf zookeeper-3.4.6.tar.gz
    cd zookeeper-3.4.6
    cp conf/zoo_sample.cfg conf/zoo.cfg
    ./bin/zkServer.sh start
    
# 启动druid服务
启动zk后，我们就可以启动druid的服务了。 首先进入到druid0.9.1.1的根目录，执行

    bin/init

druid会自动创建一个var目录， 内含俩个目录，一个是druid， 用于存放本地环境下hadoop的临时文件，索引日志，segments文件及缓存 和 任务的临时文件。 另一个是tmp用于存放其他临时文件。

接下来就可以在控制台启动druid服务了。 在单机情况下，我们可以在一台机器上启动所有的druid服务进程，分终端进行。 在分布式生产集群的环境下， druid的服务进程同样也可以在一起启动。

    java `cat conf-quickstart/druid/historical/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/historical:lib/*" io.druid.cli.Main server historical
    java `cat conf-quickstart/druid/broker/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/broker:lib/*" io.druid.cli.Main server broker
    java `cat conf-quickstart/druid/coordinator/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/coordinator:lib/*" io.druid.cli.Main server coordinator
    java `cat conf-quickstart/druid/overlord/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/overlord:lib/*" io.druid.cli.Main server overlord
    java `cat conf-quickstart/druid/middleManager/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/middleManager:lib/*" io.druid.cli.Main server middleManager

druid服务进程启动后，可以在控制台看到相应的日志信息。

我前一篇文章中提到过druid有几种节点， 上面的启动命令，对应的就是druid的各种节点

* historical 为Historical Nodes节点进程。主要用于查询时从deepstroage 加载segments。
* broker 为Broker Nodes 节点进程。 主要为接收客户端任务，任务分发，负载，以及结果合并等。
* coordinator 为 Coordinator Nodes 节点进程。 主要负责segments的管理和分发。
* ``overlord`` 为 Overload Nodes 节点进程。 ``middleManager`` 为 MiddleManager Nodes 节点进程。 overload 和 middleManager是创建索引的主要服务进程， 具体会在接下来的章节中详细介绍

如果想关闭服务，直接在控制台ctrl + c 就可以了。 如果你彻底清理掉之前的内容，重新开始，需要在关闭服务后，删除目录下的var 文件， 重新执行init脚本。


# 批量加载数据

服务启动之后，我们就可以将数据load到druid中进行查询了。在druid0.9.1.1的安装包中，自带了2015-09-12的wikiticker数据。我们可以用此数据来作为我们druid的学习实例。

首先我们看一下wikipedia的数据， 除了时间之外，包含的维度(dimensions)有：

* channel
* cityName
* comment
* countryIsoCode
* countryName
* isAnonymous
* isMinor
* isNew
* isRobot
* isUnpatrolled
* metroCode
* namespace
* page
* regionIsoCode
* regionName
* user

度量(measures) 我们可以设置如下

* count
* added
* deleted
* delta
* user_unique

确定了度量，维度之后，接下来我们就可以导入数据了。首先，我们需要向druid提交一个注入数据的任务，并将目录指向我们需要加载的数据文件wikiticker-2015-09-12-sampled.json

Druid是通过post请求的方式提交任务的， 上面我们也讲过，overload node 用于数据的加载，所以需要在overload节点上执行post请求， 目前单机环境，无需考虑这个。

在druid根目录下执行

    curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/wikiticker-index.json localhost:8090/druid/indexer/v1/task

其中``wikiticker-index.json`` 文件指明了数据文件的位置，类型，数据的schema(如度量，维度，时间，在druid中的数据源名称等)等信息， 之后我也会详细的介绍，大家也可以从官网上查

当控制台打印如下信息后，说明任务提交成功

    {"task":"index_hadoop_wikipedia_2013-10-09T21:30:32.802Z"}

可以在overload控制台 ``<http://localhost:8090/console.html>``来查看任务的运行情况， 当状态为“SUCCESS”时， 说明任务执行成功。

当数据注入成功后，historical node会加载这些已经注入到集群的数据，方便查询，这大概需要花费1-2分钟的时间。 你可以在coordinator 控制台``<http://localhost:8081/#/>``来查看数据的加载进度

当名为wikiticker的datasource 有个蓝色的小圈，并显示fully available时，说明数据已经可以了。可以执行查询操作了。

# 加载流数据
为了实现流数据的加载，我们可以通过一个简单http api来向druid推送数据，而tranquility就是一个不错的数据生产组件

下载并安装tranquility

    curl -O [http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.0.tgz](http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.0.tgz)
    tar -xzf tranquility-distribution-0.8.0.tgz
    cd tranquility-distribution-0.8.0

druid目录中自带了一个配置文件 ``conf-quickstart/tranquility/server.json`` 启动tranquility服务进程， 就可以向druid的 metrics datasource 推送实时数据。

    bin/tranquility server -configFile <path_to_druid_distro>/conf-quickstart/tranquility/server.json

这一部分向大家介绍了如何通过tranquility服务来加载流数据， 其实druid还可以支持多种广泛使用的流式框架， 包括Kafka, Storm, Samza, and Spark Streaming等

流数据加载中，维度是可变的，所以在schema定义的时候无需特别指明维度，而是将数据中任何一个字段都当做维度。而该datasource的度量则包含

* count
* value_sum (derived from `value` in the input)
* value_min (derived from `value` in the input)
* value_max (derived from `value` in the input)

我们采用了一个脚本，来随机生成度量数据，导入到这个datasource中

    bin/generate-example-metrics | curl -XPOST -H'Content-Type: application/json' --data-binary @- [http://localhost:8200/v1/post/metrics](http://localhost:8200/v1/post/metrics)

执行完成后会返回
 
    {"result":{"received":25,"sent":25}}

这表明http server 从你这里接收到了25条数据，并发送了这25条数据到druid。 在你第一次运行的时候，这个过程需要花一些时间，一段数据加载成功后，就可以查询了

接下来就是数据查询了，我们可以采用如下几种方式来查询数据

## Direct Druid queries 直接通过druid查询

druid提供了基于json的富文本查询方式。在提供的示例中，``quickstart/wikiticker-top-pages.json`` 是一个topN的查询实例

    curl -L -H'Content-Type: application/json' -XPOST --data-binary @quickstart/wikiticker-top-pages.json [http://localhost:8082/druid/v2/?pretty](http://localhost:8082/druid/v2/?pretty)

## **Visualizing data 数据可视化**

druid是面向用户分析应用的完美方案， 有很多开源的应用支持druid的数据可视化， 如pivot, caravel 和 metabase等

## **SQL and other query libraries 查询组件**

有许多查询组件供我们使用，如sql引擎， 还有其他各种语言提供的组件，如python和ruby。 具体如下：

+ python： [druid-io/pydruid](https://github.com/druid-io/pydruid)
+ R: [druid-io/RDruid](https://github.com/druid-io/RDruid)
+ JavaScript: [implydata/plywood](https://github.com/implydata/plywood)
+ [7eggs/node-druid-query](https://github.com/7eggs/node-druid-query)
+ Clojure: [y42/clj-druid](https://github.com/y42/clj-druid)
+ Ruby: [ruby-druid/ruby-druid](https://github.com/ruby-druid/ruby-druid)
+ [redBorder/druid_config](https://github.com/redBorder/druid_config)
+ SQL: [Apache Calcite](http://calcite.apache.org/)
+ [implydata/plyql](https://github.com/implydata/plyql)
+ PHP: [pixelfederation/druid-php](https://github.com/pixelfederation/druid-php)


