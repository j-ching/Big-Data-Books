## Spark Overview

apache spark 是一个快速的，分布式计算系统。 它提供了包括java， scala， python和R等多种API，其优化的计算引擎支持图计算。 在其基础上，它还提供了``Spark SQL``来处理结构化框架， ``MLlib``处理机器学习的相关问题， ``GraphX`` 用于图计算以及``Spark Streaming``用于流式处理。

## Running the Examples and Shell

运行java或者scala的例子，可以使用``bin/run-example <class>`` 

    ./bin/run-example SparkPi 10

你也可以直接通过spark shell来运行

    ./bin/spark-shell --master local[2]

--master 用于指明集群中master的url，本地运行的时候，为``local``,中括号中指定线程数

