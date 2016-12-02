## Spark Overview

apache spark 是一个快速的，分布式计算系统。 它提供了包括java， scala， python和R等多种API，其优化的计算引擎支持图计算。 在其基础上，它还提供了``Spark SQL``来处理结构化框架， ``MLlib``处理机器学习的相关问题， ``GraphX`` 用于图计算以及``Spark Streaming``用于流式处理。

## Running the Examples and Shell

运行java或者scala的例子，可以使用``bin/run-example <class>`` 

    ./bin/run-example SparkPi 10

你也可以直接通过spark shell来运行

    ./bin/spark-shell --master local[2]

--master 用于指明集群中master的url，本地运行的时候，为``local``,中括号中指定线程数

Spark同时提供了python的API， 运行python API, 使用``bin/pyspark``
 
    ./bin/pyspark --master local[2]

example同样也提供了python版， 如

    ./bin/spark-submit examples/src/main/python/pi.py 10

从1.4开始，spark同样提供了R API， 运行R API， 使用 ``bin/sparkR``

    ./bin/sparkR --master local[2]

R实现的example也同样实现了， 如

    ./bin/spark-submit examples/src/main/r/dataframe.R

## Launching on a Cluster

spark可以自己运行，也可以构建在一些已经存在的集群管理框架上， 当前支持的集群框架有：

+ **Amazon EC2** 
+ **Standalone Deploy Mode**
+ **Apache Mesos**
+ **Hadoop YARN**