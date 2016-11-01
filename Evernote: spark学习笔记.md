title: spark 笔记
notebook: 技术相关
tags: spark

[TOC]

## maven依赖

	<dependency>
		<groupId>org.apache.spark</groupId>
		<artifactId>spark-core_${scala.tools.version}</artifactId>
		<version>1.6.0</version>
	</dependency>

## 初始化spark

	val conf = new SparkConf().setAppName(appName).setMaster(master)
	new SparkContext(conf)


## 应用提交

	./bin/spark-submit \
	  --class <main-class>
	  --master <master-url> \
	  --deploy-mode <deploy-mode> \
	  --conf <key>=<value> \
	  ... # other options
	  <application-jar> \
	  [application-arguments]


####  Run application locally on 8 cores

	./bin/spark-submit \
	  --class org.apache.spark.examples.SparkPi \
	  --master local[8] \
	  /path/to/examples.jar \
	  100

#### Run on a Spark standalone cluster in client deploy mode

	./bin/spark-submit \
	  --class org.apache.spark.examples.SparkPi \
	  --master spark://207.184.161.138:7077 \
	  --executor-memory 20G \
	  --total-executor-cores 100 \
	  /path/to/examples.jar \
	  1000

####  Run on a Spark standalone cluster in cluster deploy mode with supervise

	./bin/spark-submit \
	  --class org.apache.spark.examples.SparkPi \
	  --master spark://207.184.161.138:7077 \
	  --deploy-mode cluster
	  --supervise
	  --executor-memory 20G \
	  --total-executor-cores 100 \
	  /path/to/examples.jar \
	  1000


####  Run on a YARN cluster

	export HADOOP_CONF_DIR=XXX
	./bin/spark-submit \
	  --class org.apache.spark.examples.SparkPi \
	  --master yarn \
	  --deploy-mode cluster \  # can be client for client mode
	  --executor-memory 20G \
	  --num-executors 50 \
	  /path/to/examples.jar \
	  1000

#### Run a Python application on a Spark standalone cluster

	./bin/spark-submit \
	  --master spark://207.184.161.138:7077 \
	  examples/src/main/python/pi.py \
	  1000

#### Run on a Mesos cluster in cluster deploy mode with supervise

	./bin/spark-submit \
	  --class org.apache.spark.examples.SparkPi \
	  --master mesos://207.184.161.138:7077 \
	  --deploy-mode cluster
	  --supervise
	  --executor-memory 20G \
	  --total-executor-cores 100 \
	  http://path/to/examples.jar \
	  1000


## 关闭任务

关闭spark应用可以调用SparkContext的stop() 方法或者直接退出应用 System.exit(0) 或者 sys.exit()

## spark操作类型

RDD的操作可以分为转化操作和行为操作， 转化操作生成新的RDD，行为操作会真正的执行计算

## 数据持久化

默认情况下，spark的RDD会在每次对他们进行行为操作的时候重新计算，如果想在多个行为操作中重用同一个RDD，可以通过RDD.persist() 让Spark把这个RDD缓存起来。默认的在第一次对持久化的RDD计算之后，spark会把RDD的内容保存在内存中。我们也可以把RDD保存在其他存储中。
cache() 与使用默认存储级别调用persist()是一样的，即保存在内存中。

## parallelize数据加载

parallelize() 把程序中一个已有的集合传给SparkContext

## 常用的行为操作

行为操作中， count() 返回计算结果数， take() 来收集RDD的一些元素， collection() 可以用来获取整数个RDD的元素, 只有当你的整个集群数据集能在单台机器的内存中放的下的时候，才能使用collect()。 除此之外，可以使用saveAsTExtFile() saveAsSequenceFile() 或者其他行为操作把数据保存下来。

## 转化操作

   + map(func) 接收一个函数，把这个函数用于RDD中的每个元素，将函数的返回结果作为结果RDD中的对应元素的值
   + filter(func) 接收一个函数，并将RDD中满足该函数的元素放入新的RDD中返回
   + flatmap(func) 接收一个函数，将这个函数用于RDD的每个元素上，不过返回的不是一个元素，而是一个返回值序列的迭代器， 将返回的迭代器拍扁
   + mapPartitions(func) 与map相同，接收一个函数，将这个函数用于RDD的每个分区上， 函数应该是``` Iterator<T> => Iterator<U>```
   + mapPartitionsWithIndex(func) 与mapPartitions相同， 接收一个函数用于RDD的每个分区上，函数应该是 ```(Int, Iterator<T>) => Iterator<U>```
   + distinct([numTasks]) 生成一个只包含不同元素的新RDD， 开销较大，因为需要将所有数据通过网络进行混洗
   + union(other)  返回一个包含俩个RDD的所有元素的新的RDD， 会包含重复的数据
   + intersection(other) 只返回俩个RDD都有的元素，会去掉所有重复的元素， 性能较差，因为也需要通过网络来混洗数据
   + subtract(other) 返回只存在于第一个RDD而不存在第二个RDD的所有元素的新的RDD， 性能较差，也需要通过网络混洗数据
   + cartesian(other) 返回笛卡尔集
   + sample(withReplacement, fraction, seed) 对RDD采样。
   + groupByKey([numTasks]) 用于数据集 ```(K, V)```上，返回数据集``` (K, Iterable<V>) ```
   + reduceByKey(func, [numTasks]) 对数据集 ```(K, V)``` 进行计算，返回```(K, Iterable<V>) ```
   + aggregateByKey(zeroValue)(seqOp, combOp, [numTasks])
   + sortByKey([ascending], [numTasks])
   + join(otherDataset, [numTasks])
   + cogroup(otherDataset, [numTasks])
   + pipe(command, [envVars])
   + coalesce(numPartitions)
   + repartition(numPartitions)
   + repartitionAndSortWithinPartitions(partitioner)


## 行为操作

   + reduce(func)
   + colect()  返回RDD中的所有元素
   + count() 返回RDD中元素的个数
   + countByKey()
   + countByValue() 返回各个元素在RDD中出现的次数
   + take(n)	从RDD中返回n个元素
   + top(n)   从RDD中返回前面n个元素
   + first()
   + takeOrderd(n)(ordering) 从RDD中按照提供的殊勋返回最前面的n个元素
   + takeSample 从RDD中返回任意一些元素
   + reduce(func) 接收一个函数作为参数，这个函数要操作俩个相同元素类型的RDD数据并返回一个同样类型的新元素，可以求元素的总和，元素个数以及其他类型的聚合操作
   + fold(zero)(func) 与reduce一样，但需要提供初始值
   + aggregate()  与reduce一样，但是返回不同类型的函数
   + foreach(func) 对RDD中的每个元素使用给定的函数
   + saveAsTextFile(path)
   + saveAsSequenceFile(path)
   + saveAsObjectFile(path)


## 持久化
   persist()调用不会触发强制求值
   scala和java 默认会把数据以序列化的形式缓存在jvm的堆空间中。
   当我们让spark持久化存储一个RDD的时候，计算出RDD的节点会分别保存他们所求的分区数据，如果一个有持久化数据的节点发生故障，spark会在需要用到混粗的数据时重新计算丢失的数据分区。如果希望节点故障的情况不会拖累我们的执行速度，也可以把数据被飞到多个节点上。

   org.apache.spark.storage.StorageLevel

   + MEMORY_ONLY
   + MEMORY_ONLY_SER
   + MEMORY_AND_DISK
   + MEMORY_AND_DISK_SER
   + DISK_ONLY

   如果缓存的数据太多，内存中放不下，spark自动利用最近最少使用（LRU）的缓存策略把最老的分区从内存中移除。对于仅把数据存放在内存的缓存级别，下一次要用到已经被移除的分区时，这些分区就需要重新计算。 但是对于使用使用内存与磁盘的缓存级别分区来说，被移除的分区都会写入磁盘，不论哪一种情况都不必担心你的作业因为缓存了太多数据而被打断。不过缓存不要的数据会导致有用的数据被移除内存，带来更多重算的时间开销。

## upersist()
手动把持久化的RDD从缓存移除

## stats()
打印RDD的总数，平均数，最大值，最小值等

## span()
用第一个特定的符号将一行拆分为俩部分

##lookup()
查找数据
