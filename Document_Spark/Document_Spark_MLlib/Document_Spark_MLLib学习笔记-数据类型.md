title: spark MLlib 学习笔记-DataType
notebook: 技术相关
tags: spark

[TOC]

MLlib 是spark的机器学习库， 包含了很多公用的学习算法和组件，包括classification(分类), regression(回归), clustering(聚类), collaborative filtering(协同过滤), dimensionality reduction(降维)，也有一些低级别的原语以及高级别的管道API
提供俩个packages:

+ spark.mllib 提供了RDD级别的原生的API
+ spark.ml 提供了DataFrames级别的高级的API来构造ml管道

推荐使用spark.ml包，因为DataFrames API 更灵活和多样化。但我们在开发spark.ml的同时开发spark.mllib.

# Data Types - MLlib

mllib 单机存储的本地向量和矩阵，同样也支持分布式存储的矩阵。 下面的算数运算时[Breeze](http://www.scalanlp.org/)和[jblas](http://jblas.org/)提供的。 学习管理中的训练样本在MLlib中被称作标签点。

### Local vector

MLlib支持俩种类型的向量： 密集向量和稀疏向量

+ 密集向量： 用二维数组来标示他的值
+ 稀疏向量： 用俩个平行的数组标示: 指标和值

如： 一个向量(1.0, 0.0, 3.0) 可以表示为如下俩种形式
密集矩阵表示为 [1.0, 0.0, 3.0]
稀疏矩阵表示为 (3, [0,2], [1.0, 3.0]) 3是向量元素个数， 第一个数组为数据的索引， 第二个数组为元素

	import org.apache.spark.mllib.linalg.{Vector, Vectors}

	// Create a dense vector (1.0, 0.0, 3.0).
	val dv: Vector = Vectors.dense(1.0, 0.0, 3.0)
	// Create a sparse vector (1.0, 0.0, 3.0) by specifying its indices and values corresponding to nonzero entries.
	val sv1: Vector = Vectors.sparse(3, Array(0, 2), Array(1.0, 3.0))
	// Create a sparse vector (1.0, 0.0, 3.0) by specifying its nonzero entries.
	val sv2: Vector = Vectors.sparse(3, Seq((0, 1.0), (2, 3.0)))


### Labeled point

标签页是一个本地的向量， 不管是密集和是稀疏，它将标签和数据连接起来。  在MLlib中，标签向量被用在机器学习管理中。 我们使用二维向量存储标签， 标签可以用于回归和分类中。 对于二分类问题，标签非0及1，对多分类问题，标签可以是0,1,2.....

	import org.apache.spark.mllib.linalg.Vectors
	import org.apache.spark.mllib.regression.LabeledPoint

	// Create a labeled point with a positive label and a dense feature vector.
	val pos = LabeledPoint(1.0, Vectors.dense(1.0, 0.0, 3.0))

	// Create a labeled point with a negative label and a sparse feature vector.
	val neg = LabeledPoint(0.0, Vectors.sparse(3, Array(0, 2), Array(1.0, 3.0)))

##### 稀疏数据
MLlib支持从LIBSVM格式读取训练样本，这些数据默认由[LIBSVM](http://www.csie.ntu.edu.tw/~cjlin/libsvm/) 和 [LIBLINEAR](http://www.csie.ntu.edu.tw/~cjlin/liblinear/) 格式化, 每一行都是如下的文本格式

	label index1:value1 index2:value2 ...

MLUtils.loadLibSVMFile 读取LIBSVM格式的寻列样本

	import org.apache.spark.mllib.regression.LabeledPoint
	import org.apache.spark.mllib.util.MLUtils
	import org.apache.spark.rdd.RDD

	val examples: RDD[LabeledPoint] = MLUtils.loadLibSVMFile(sc, "data/mllib/sample_libsvm_data.txt")


### Local matrix

MLlib支持稠密矩阵和稀疏矩阵。稠密矩阵将整个数据值保存在单独的一个列顺序的二维数组中， 稀疏矩阵则将数据按CSC格式存储在列顺序的非零数组中。

	import org.apache.spark.mllib.linalg.{Matrix, Matrices}

	// Create a dense matrix ((1.0, 2.0), (3.0, 4.0), (5.0, 6.0))
	val dm: Matrix = Matrices.dense(3, 2, Array(1.0, 3.0, 5.0, 2.0, 4.0, 6.0))

	// Create a sparse matrix ((9.0, 0.0), (0.0, 8.0), (0.0, 6.0))
	val sm: Matrix = Matrices.sparse(3, 2, Array(0, 1, 3), Array(0, 2, 1), Array(9, 6, 8))

### Distributed matrix

分布式矩阵包括行指标， 列指标和二维的数据值数组，分布的保存在一个或者多个RDD上。 选择合适的存储格式来保存大量的分布式矩阵是很重要的。 将一个分布式矩阵转为不同的格式，需要全局的清晰，消耗高。
目前有三种分布式矩阵格式的实现。

典型的是```RowMatrix```, RowMatrix是基于行的分布式矩阵，单没有维护行指标， 如一个特征向量集合。 它是基于行的RDD， 每一样是一个本地向量。 我们假定RowMatrix的列数不大， 所以一个单独的本地向量可以和驱动程序通讯，也能被保存在单个节点上。 ``` IndexedRowMatrix ``` 与RowMatrix 相似 ， 但是包含行指标的，所以可以很好的识别行以及执行jsons操作。 ``CoordinateMatrix``` 是以COO格式保存的一个分布式矩阵， 它的全部是一个RDD

+ RowMatrix
+ IndexedRowMatrix
+ CoordinateMatrix

##### RowMatrix

RowMatrix可以从RDD[Vector]实例创建， 然后我们就可以对列做统计和分解了。 [QR](https://en.wikipedia.org/wiki/QR_decomposition)分解法的公式为A=QR, Q是一个正交矩阵， R是一个上三角矩阵。 [SVD](https://en.wikipedia.org/wiki/Singular_value_decomposition) 和 [PCA](https://en.wikipedia.org/wiki/Principal_component_analysis) 用来降维

	import org.apache.spark.mllib.linalg.Vector
	import org.apache.spark.mllib.linalg.distributed.RowMatrix

	val rows: RDD[Vector] = ... // an RDD of local vectors
	// Create a RowMatrix from an RDD[Vector].
	val mat: RowMatrix = new RowMatrix(rows)

	// Get its size.
	val m = mat.numRows()
	val n = mat.numCols()

	// QR decomposition
	val qrResult = mat.tallSkinnyQR(true)

##### IndexedRowMatrix

	import org.apache.spark.mllib.linalg.distributed.{IndexedRow, IndexedRowMatrix, RowMatrix}

	val rows: RDD[IndexedRow] = ... // an RDD of indexed rows
	// Create an IndexedRowMatrix from an RDD[IndexedRow].
	val mat: IndexedRowMatrix = new IndexedRowMatrix(rows)

	// Get its size.
	val m = mat.numRows()
	val n = mat.numCols()

	// Drop its row indices.
	val rowMat: RowMatrix = mat.toRowMatrix()

##### CoordinateMatrix

coordinateMatrix是一个分布式矩阵，基于自己本身的RDD。 每一个实例是一个tuple ```(i: Long, j: Long, value: Double)```, i 是行标，j是列标，value为值。 CoordinateMatrix用于维度特别多，且矩阵很稀疏。

	import org.apache.spark.mllib.linalg.distributed.{CoordinateMatrix, MatrixEntry}

	val entries: RDD[MatrixEntry] = ... // an RDD of matrix entries
	// Create a CoordinateMatrix from an RDD[MatrixEntry].
	val mat: CoordinateMatrix = new CoordinateMatrix(entries)

	// Get its size.
	val m = mat.numRows()
	val n = mat.numCols()

	// Convert it to an IndexRowMatrix whose rows are sparse vectors.
	val indexedRowMatrix = mat.toIndexedRowMatrix()

##### BlockMatrix

BlockMatrix也是一个分布式矩阵，基于MatrixBlocks的RDD， 它的tuple为 ``` ((Int, Int), Matrix)```, (Int, Int)为block的索引， Matrix是``rowsPerBlock x colsPerBlock```个数的子矩阵。

A BlockMatrix can be most easily created from an IndexedRowMatrix or CoordinateMatrix by calling toBlockMatrix. toBlockMatrix creates blocks of size 1024 x 1024 by default. Users may change the block size by supplying the values through toBlockMatrix(rowsPerBlock, colsPerBlock).

	import org.apache.spark.mllib.linalg.distributed.{BlockMatrix, CoordinateMatrix, MatrixEntry}

	val entries: RDD[MatrixEntry] = ... // an RDD of (i, j, v) matrix entries
	// Create a CoordinateMatrix from an RDD[MatrixEntry].
	val coordMat: CoordinateMatrix = new CoordinateMatrix(entries)
	// Transform the CoordinateMatrix to a BlockMatrix
	val matA: BlockMatrix = coordMat.toBlockMatrix().cache()

	// Validate whether the BlockMatrix is set up properly. Throws an Exception when it is not valid.
	// Nothing happens if it is valid.
	matA.validate()

	// Calculate A^T A.
	val ata = matA.transpose.multiply(matA)
