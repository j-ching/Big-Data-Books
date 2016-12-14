title: Spark Stream学习笔记
notebook: 技术相关
tags: spark

[TOC]

# Overview

Spark Streaming 是spark 核心API的一个扩展，是一个可扩展，高吞吐，高容错的实时数据流处理框架。 数据来源可以有多种，包括Kafka，Flume，ZeroMQ， Kinesis或者TCP sockets， 数据经过一系列的复杂的算法处理后(如map，reduce，join或者window)，将结果数据写入到文件系统，数据库或者实时的dashboards上。 事实上，spark的machine learning和graph processing 算法都可以运行在data stream上。

![spark streaming 架构](http://spark.apache.org/docs/1.6.0/img/streaming-arch.png)

SparkStream接收到输入的数据流之后， 将数据流分割为多个batch， 交给Spark引擎来技术，最后按batch 生成流式的结果集

![spark Streaming 内部流结构](http://spark.apache.org/docs/1.6.0/img/streaming-flow.png)

Spark Streaming 提供了一个抽象的概念，叫做discretized stream 或者DStream， 相当于连续的数据流。 Dstream 可以从Kafka， flume以及Kinesis等数据源来创建，也可以通过其他Dstream 经过一系列的操作获取。 DStream可以被理解为一系列的RDD

#A Quick Example
首先我们来看一下，一个简单的spark streaming程序是什么样子的。 举个例子，我们想统计一下，从TCP端口过来的文本数据，每个单词出现的字数。 我们创建一个本地的StsreamingContext，拥有俩个线程， 设置每1s发送一批

    import org.apache.spark._
    import org.apache.spark.streaming._
    import org.apache.spark.streaming.StreamingContext._

    val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCout")
    val ssc = new StreamingContext(conf, Seconds(1))

使用这个context， 我们可以从一个指定hostname和port的tcp数据源中读取DStream

    val lines = ssc.socketTextStream("localhost", 9999)

lines是Dstream， 表示从数据服务器接收到的数据流。Dstream中的每一条记录都都是一条文本，然后我们把这些记录通过空格分隔为单词

    val words = lines.flatMap(_.split(" "))

flatMap是一个一边多Dstream的操作， 将源Dstream的每一条数据转为多条新的记录。 在此处，每一条记录都被分隔为多个单词，单词流被表示为单词Dstream。 然后我们就去数这些单词。

    import org.apache.spark.streaming.StreamingContext._
    
    val pairs = words.map(word => (word, 1))
    val wordCounts = pairs.reduceByKey(_ + _)

    wordCounts.print()
    
words Dstream 进一步映射成为一个由(word，1)键值对组成的Dstream， 然后通过reduce获取每一批中单词的的平率， 最后，通过workCounts.print() 每一秒打印一次结果。

当这些代码被执行后， Spark Streaming 只是对计算做了准备，真正的处理过程还没有开始， 当所有的转换完成之后， 计算过程才会开始，我们最后调用

    ssc.start()
    ssc.awaitTermination()

该段代码可以在Spark Streaming Example的NetworkWordCount找到
如果你已经下载并编译了spark， 可以按如下的步骤去执行。
首先你需要运行一个netcat(unix系统中一个小的组件)， 

    $ nc -lk 9999

然后在另一个终端，执行example代码

    $ ./bin/run-example streaming.NetworkWordCount localhost 9999

然后，在netcat终端输入的任何字符， 都会被记录被打印出来

# Basic Concepts

## 相关连接
与spark相同，spark streaming同样可以使用Maven Central 来管理。 开发自己的spark Streaming 程序时，需要将如下的依赖添加到你的sbt或者maven工程中

    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-streaming_2.10</artifactId>
        <version>1.6.0</version>
    </dependency>

在spark Streaming的核心API中，并没有包括像kafka，flume，和kinesis等数据源的依赖，我们需要额外的添加相关的依赖，类似于 spark-streaming-xyz_2.10 等

Source |   Artifact
-------|-----------
Kafka  | spark-streaming-kafka_2.10
Flume  | spark-streaming-flume_2.10
Kinesis| spark-streaming-kinesis-asl_2.10[Amazon Software License]
Twitter| spark-streaming-twitter_2.10
ZeroMQ | spark-streaming-zeromq_2.10
MQTT   | spark-streaming-mqtt_2.10


## 初始化streamingcontext

初始化Spark Streaming程序的时候，streamingContext 需要被创建，以此作为所有SparkStreaming的入口
 
StreamingContext 可以通过sparkconf实例来创建

    import org.apache.spark._
    import org.apache.spark.streaming._

    val conf = new SparkConf().setAppName(appName).setMaster(master)
    val ssc = new StreamingContext(conf, Second(1))

参数appName是你在cluster UI展现的名字. master 是一个Spark、Mesos 或者Yarn 集群的url，或者是在本地模式下的一个特殊的"local[*]". 事实上，在集群上运行程序的时候，并不需要硬编码master在程序里，而可以通过``spark-submit``来接收他。同时可以通过``ssc.sparkContext``来获取SparkContext对象

batch的时间间隔需要基于你应用所需要的频率以及集群的资源来设定，查看[Performance Tuning](http://spark.apache.org/docs/1.6.0/streaming-programming-guide.html#setting-the-right-batch-interval)查看更多细节

StreamingContext也可以从一个SparkContext来创建

    import org.apache.spark.streaming._
    
    val sc = 
    val ssc = new StreamingContext(sc, Seconds(1))

当context被创建后，你就可以做如下的操作：

1. 创建``input Dstreams``来定义数据源
2. 通过提交对``DStreams``的转化和输出操作来定义流式计算
3. 使用``streamingContext.start()`` 来启动数据的接收和处理工作
4. 使用``streamingContext.awaitTermination()``等待处理工作的结束
5. 也可以通过``streamingContext.stop()`` 来手动停止处理工作

关键点：
+ 一旦context被启动，新的流计算操作就无法被设置或者添加
+ 一旦context被停止， 就无法重新启动
+ 一个jvm钟，同一时间只能由一个streamingContext运行
+ streamingContext的stop()操作同时也关闭了SparkContext。如果只需关闭SparkContext， stop有个参数叫做``stopSparkContext``, 设置为``false``
+ SparkContext可以创建多个StreamingContext, 当前一个StreamingContext被stop(但是SparkContext 没有被stop)，下一个StreamingContext会被创建

# Discretized Streams(DStreams)
Discretized Stream或者叫DStream 是spark Streaming的一个基本的概念，你可以把它看做是连续的数据流，无论是从数据源接收的数据，还是有输入流转化处理后的数据。具体的说，一个DStream表现为一系列的RDD，每个RDD包含着指定时间间隔的一系列数据

![Dstreams](http://spark.apache.org/docs/1.6.0/img/streaming-dstream.png)

任何基于Dstream的操作，都是转为对于RDD的操作。如之前的例子，将多行文字刘转为单词，``flatMap``操作会被提交到Dstream中的每一个RDD上，然后生成单词DStream的RDD，具体流程如下：

![DStreamtranslate](http://spark.apache.org/docs/1.6.0/img/streaming-dstream-ops.png)

RDD的转化由Spark引擎来来完成。DStream的操作隐藏了这些细节，对外提供了用于交互的高级API供开发者使用。

# Input DStreams And Receivers
InputDStream 表示从输入流数据接收过来的数据流。 在之前的例子中，从netcat server接收的行数据就是InputDStream。 每个InputDStream都与一个receiver Object相关(文件流除外，后面会专门介绍)， receiver Object接收数据源的数据，并保存在内存中，供后续处理

spark Streaming 提供了俩种类型构建输入流的方式
+ **基础数据源**StreamingContext API可以直接使用的数据源，如 文件系统，socket连接 以及 akka actors
+ **高级数据源**需要使用扩展组件来访问的源， 如kafka, flume, kinesis，twitter等。

如果在你的应用中需要并行接收多种数据源，则需要创建``multiple input DStreams``. 创建的multiple receivers会同时接收多种数据流。 要注意的是，spark的worker/executor是会一直运行的，它会在spark Streaming 应用分配的core中执行， 所以确保有足够多的core来接收数据，同时运行receivers是很重要的。

有俩点：
+ 本地运行时，master url 不要使用 local 或者 local[1]. 因为这样意味着只有一个线程来运行任务。 如果你使用了一个包含receiver的Input DStream，单一的线程就会被占用来接收数据， 没有多余的线程来处理数据了。所以在本地执行时，设置master url 为 local[n], n需要比receivers的数量要多。
+ 在集群上运行时，分配给spark Streaming 应用的core数目需要比receivers要多，否则只能接收数据，没有线程处理数据

## 基础数据源
前面的示例我们已经看到了通过``ssc.socketTextStream()``从TCP soket连接中接收数据，创建DStream。除此之外， StreamingContext Api也提供了从文件和akka  actor作为数据源创建DStream。

+ **File Streams** 利用HDFS API(HDFS,S3,NFS等) 从文件系统中读取文件数据， DStream 创建如下： ``streamingContext.fileStream[KeyClass, valueClass, InputFormatClass](dataDirectory)``spark streaming将监控 dataDirectory 目录下的所有文件(嵌套目录的文件不被监控), 注意：
 + 目录下的文件需要有统一的数据格式
 + 文件必须是是通过移到目录下或者在该目录下重命名来创建
 + 一旦移动，文件就不能改变了， 如果文件持续append， 新的数据也不会被读取

