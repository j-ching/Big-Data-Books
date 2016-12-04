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




















