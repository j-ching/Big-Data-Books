---
title: spark MLLib学习笔记-基础统计
notebook: 技术相关
tags:spark
---

# Summary statistics

RDD[Vector] 提供了colStats方法在统计， 它返回 [MultivariateStatisticalSummary](http://spark.apache.org/docs/1.6.0/api/scala/index.html#org.apache.spark.mllib.stat.MultivariateStatisticalSummary)类型，包括列方式的max最大值，min最小值，mean平均值,variance方差值，非零值个数以及总数


	import org.apache.spark.mllib.linalg.Vector
	import org.apache.spark.mllib.stat.{MultivariateStatisticalSummary, Statistics}

	val observations: RDD[Vector] = ... // an RDD of Vectors

	// Compute column summary statistics.
	val summary: MultivariateStatisticalSummary = Statistics.colStats(observations)
	println(summary.mean) // a dense vector containing the mean value for each column
	println(summary.variance) // column-wise variance
	println(summary.numNonzeros) // number of nonzeros in each column

# Correlations(相关性)

在统计计算中，计算俩组数据的相关性是比较常见的。Statistics 提供了计算相关性的方法， 输入类型为俩个RDD[Double]或者一个RDD[Vector],输出为一个double类型或者一个相关性矩阵

	import org.apache.spark.SparkContext
	import org.apache.spark.mllib.linalg._
	import org.apache.spark.mllib.stat.Statistics

	val sc: SparkContext = ...

	val seriesX: RDD[Double] = ... // a series
	val seriesY: RDD[Double] = ... // must have the same number of partitions and cardinality as seriesX

	// compute the correlation using Pearson's method. Enter "spearman" for Spearman's method. If a 
	// method is not specified, Pearson's method will be used by default. 
	val correlation: Double = Statistics.corr(seriesX, seriesY, "pearson")

	val data: RDD[Vector] = ... // note that each Vector is a row and not a column

	// calculate the correlation matrix using Pearson's method. Use "spearman" for Spearman's method.
	// If a method is not specified, Pearson's method will be used by default. 
	val correlMatrix: Matrix = Statistics.corr(data, "pearson")

1.6支持的相似度有

+ pearson 皮尔逊相关系数
+ spearman 

# Stratified sampling(分层抽样)
分层抽样的方法有sampleByKey 和 sampleByKeyExact， 可很好的而运行在KV对的RDD上。 举例来说， key可以看做是标记， value看做是属性。 例如： key 可以使男/女， 文档ID，而value可以是人们的年龄或者是文档的单词

+ sampleByKey 用扔硬币的方式来决定观察者是否为样本， 因此需要一个数据的通行证以及希望的样本数量
+ sampleByKeyExact 在每一层使用sampleBykey进行简单抽样时需要更多的源， 但是会提供9准确率达9.99%的样本， 目前python不支持

sampleByKeyExact()    ⌈fk⋅nk⌉∀k∈K

	import org.apache.spark.SparkContext
	import org.apache.spark.SparkContext._
	import org.apache.spark.rdd.PairRDDFunctions

	val sc: SparkContext = ...

	val data = ... // an RDD[(K, V)] of any key value pairs
	val fractions: Map[K, Double] = ... // specify the exact fraction desired from each key

	// Get an exact sample from each stratum
	val approxSample = data.sampleByKey(withReplacement = false, fractions)
	val exactSample = data.sampleByKeyExact(withReplacement = false, fractions)

# Hypothesis testing(假设检验)
假设检验在统计学中是一张比较重要的工具，它能够有效的决策统计结果的重要程度，这种结果发生的几率，是否会发生。 spark.mllib当前支持皮尔逊卡方(χ2)检验，其根本思想就是在于比较理论频数和实际频数的吻合程度或拟合优度问题 
常用的假设检验方法有u—检验法、t检验法、χ2检验法(卡方检验)、F—检验法，秩和检验等。

p-value： 通过查卡方临界值表获取


	import org.apache.spark.SparkContext
	import org.apache.spark.mllib.linalg._
	import org.apache.spark.mllib.regression.LabeledPoint
	import org.apache.spark.mllib.stat.Statistics._

	val sc: SparkContext = ...

	val vec: Vector = ... // a vector composed of the frequencies of events

	// compute the goodness of fit. If a second vector to test against is not supplied as a parameter, 
	// the test runs against a uniform distribution.  
	val goodnessOfFitTestResult = Statistics.chiSqTest(vec)
	println(goodnessOfFitTestResult) // summary of the test including the p-value, degrees of freedom, 
	                                 // test statistic, the method used, and the null hypothesis.

	val mat: Matrix = ... // a contingency matrix

	// conduct Pearson's independence test on the input contingency matrix
	val independenceTestResult = Statistics.chiSqTest(mat) 
	println(independenceTestResult) // summary of the test including the p-value, degrees of freedom...

	val obs: RDD[LabeledPoint] = ... // (feature, label) pairs.

	// The contingency table is constructed from the raw (feature, label) pairs and used to conduct
	// the independence test. Returns an array containing the ChiSquaredTestResult for every feature 
	// against the label.
	val featureTestResults: Array[ChiSqTestResult] = Statistics.chiSqTest(obs)
	var i = 1
	featureTestResults.foreach { result =>
	    println(s"Column $i:\n$result")
	    i += 1
	} // summary of the test

同时spark.mlib 还提供了Kolmogorov-Smirnov (KS)检验的1-sample， 2-sided实现。 

	import org.apache.spark.mllib.stat.Statistics

	val data: RDD[Double] = ... // an RDD of sample data

	// run a KS test for the sample versus a standard normal distribution
	val testResult = Statistics.kolmogorovSmirnovTest(data, "norm", 0, 1)
	println(testResult) // summary of the test including the p-value, test statistic,
	                    // and null hypothesis
	                    // if our p-value indicates significance, we can reject the null hypothesis

	// perform a KS test using a cumulative distribution function of our making
	val myCDF: Double => Double = ...
	val testResult2 = Statistics.kolmogorovSmirnovTest(data, myCDF)

# Streaming Significance Testing(流式显著性检验)
spark.mllib 提供了一些在线的测试，像A/B testing. 这类测试运行在spark streaming DStream[(boolean, Double)],  每一个tuple的第一个元素为control group(false)或者treatment group(true), 第二个元素是观察者的值

Streaming significance testing支持俩种参数

+ peacePeriod
+ windowSize
	
	val data = ssc.textFileStream(dataDir).map(line => line.split(",") match {
	  case Array(label, value) => BinarySample(label.toBoolean, value.toDouble)
	})

	val streamingTest = new StreamingTest()
	  .setPeacePeriod(0)
	  .setWindowSize(0)
	  .setTestMethod("welch")

	val out = streamingTest.registerStream(data)
	out.print()

# Random data generation(生成随机数)
RandomRDDs 提供了一个工厂方法来生成随机的double RDD或者Vector RDD。 

	import org.apache.spark.SparkContext
	import org.apache.spark.mllib.random.RandomRDDs._

	val sc: SparkContext = ...

	// Generate a random double RDD that contains 1 million i.i.d. values drawn from the
	// standard normal distribution `N(0, 1)`, evenly distributed in 10 partitions.
	var u = RandomRDDs.normalRDD(sc, 1000L, 1)
	// Apply a transform to get a random double RDD following `N(1, 4)`.
	val v = u.map(x => 1.0 + 2.0 * x)


# Kernel density estimation(核密度估计)
简单贝叶斯分类：对于数值属性，如果不服从正态分布，但不知道服从何种分布形式，可以采用核密度估计的方法来进行预测。

	import org.apache.spark.mllib.stat.KernelDensity
	import org.apache.spark.rdd.RDD

	val data: RDD[Double] = ... // an RDD of sample data

	// Construct the density estimator with the sample data and a standard deviation for the Gaussian
	// kernels
	val kd = new KernelDensity()
	  .setSample(data)
	  .setBandwidth(3.0)

	// Find density estimates for the given values
	val densities = kd.estimate(Array(-1.0, 2.0, 5.0))
