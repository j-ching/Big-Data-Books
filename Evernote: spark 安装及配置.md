---
title: Spark Overview
notebook: 技术相关
tags:
---
Apache Spark 是一个快速的分布式计算系统，它针对java，scala, python 和 R 提供了高水平的api， 同时也提供了支持图计算的引擎。 spark还提供了一系列工具，如spark sql 用于通过sql处理结构化数据， mlib用于机器学习， graphx用于图形处理， 还有spark streaming.

# 下载

从[下载页](http://spark.apache.org/downloads.html)获取spark。 由于目前使用的是spark-1.6.0-cdh5.8.0， 我们暂时介绍spark1.6.0。 spark使用hadoop客户端来调用hdfs和yarn。预先下载目前比较流行的版本的hadoop包，通过制定spark的classpath，可以使用任何版本的hadoop。

#### Using Spark's "Hadoop Free" Build
从spark1.4开始， 项目中的"hadoop free" 允许用户非常容易的将spark与任何版本的hadoop相关联。你只需要制定 ``` SPARK_DIST_CLASSPATH ``` , 用于指向含有hadoop包的目录。大多数情况下，都是在conf/spark-env.sh 中设置的。 

	### in conf/spark-env.sh ###

	# If 'hadoop' binary is on your PATH
	export SPARK_DIST_CLASSPATH=$(hadoop classpath)

	# With explicit path to 'hadoop' binary
	export SPARK_DIST_CLASSPATH=$(/path/to/hadoop/bin/hadoop classpath)

	# Passing a Hadoop configuration directory
	export SPARK_DIST_CLASSPATH=$(hadoop --config /path/to/configs classpath)


如果你需要从源码中构建spark， 请查看[这里](http://spark.apache.org/docs/1.6.0/building-spark.html)
