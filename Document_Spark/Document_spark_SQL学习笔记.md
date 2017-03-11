[TOC]

# Overview

spark sql是spark的一个模块， 用于处理结构化得数据。 与spark rdd api 不同， spark SQl针对结构化数据提供了更多接口。spark sql 应用一些额外的信息来提升性能。 这里有一下几种方式来使用spark sql，包括sql， DataFrames API 和DataSets API。

### SQL
Spark SQL可以用于sql查询， 包括正常的SQL或者HiveQL。  Spark SQL也可以用来从已安装的hive中读取数据。 使用编程语言运行SQL， 运行结果会返回一个DataFrame。 SQL可以通过命令行方式运行或者通过JDBC/ODBC

### DataFrames
DataFrame基于列存储的分布式数据集合。 理论上等同于关系型数据库的一个table，或者R/Python中的一个数据框架， but with richer optimizations under the hood。 DataFrames 可以从多个数据源组中构建生成， 包括结构化数据文件，hive表，外部数据库以及已经存在的RDD。

### Datasets
Datasets是在spark1.6中新增的实验性接口， 意在提高RDD的优势，充分利用Spark SQl 计算引擎的性能。 Dataset 可以从JVM objects 中创建，通过转化操作(如map， flatmap， filter等)来操作。

# Getting Started

### Starting Point: SQLContext

创建sqlContext

	val sc: SparkContext // An existing SparkContext.
	val sqlContext = new org.apache.spark.sql.SQLContext(sc)

除了sqlContext之外，也可以创建hiveContext， 在sqlcontxt之上提供了更多的附加功能， 包括： 使用完整的HiveQL， 使用hive的UDF，从hive表中读取数据。  使用HiveContext， 无需安装hive， 只要数据源在SQLContext可用即可。 HiveContext 只是在spark构建的时候，打包了所有hive的依赖。如果这些依赖对你的应用来说不是一个问题，那推荐使用hiveContext。

SQL中的一些特定的变量可以通过```spark.sql.dialect```选项来设置。 在SQLContext中可以通过setConf来设置参数， 在SQL的命令行则可以通过```set key=value``` 来设置。

### Creating DataFrames

对于sqlContext， 应用可以从RDD， hive Table， 以及数据源来创建DataFrames

	val sc: SparkContext // An existing SparkContext.
	val sqlContext = new org.apache.spark.sql.SQLContext(sc)

	val df = sqlContext.read.json("examples/src/main/resources/people.json")

	// Displays the content of the DataFrame to stdout
	df.show()

### DataFrame Operations

	val sc: SparkContext // An existing SparkContext.
	val sqlContext = new org.apache.spark.sql.SQLContext(sc)

	// Create the DataFrame
	val df = sqlContext.read.json("examples/src/main/resources/people.json")

	// Show the content of the DataFrame
	df.show()
	// age  name
	// null Michael
	// 30   Andy
	// 19   Justin

	// Print the schema in a tree format
	df.printSchema()
	// root
	// |-- age: long (nullable = true)
	// |-- name: string (nullable = true)

	// Select only the "name" column
	df.select("name").show()
	// name
	// Michael
	// Andy
	// Justin

	// Select everybody, but increment the age by 1
	df.select(df("name"), df("age") + 1).show()
	// name    (age + 1)
	// Michael null
	// Andy    31
	// Justin  20

	// Select people older than 21
	df.filter(df("age") > 21).show()
	// age name
	// 30  Andy

	// Count people by age
	df.groupBy("age").count().show()
	// age  count
	// null 1
	// 19   1
	// 30   1

### Running SQL Queries Programmatically
sqlContext 可以运行sql查询，将结果作为DataFrame返回

	val sqlContext = ... // An existing SQLContext
	val df = sqlContext.sql("SELECT * FROM table")

### Creating Datasets
Datasets 与RDD很相似，但不同于Java Serialization与 kryo的序列化方式，Datasets使用一个特殊的序列化方式Encoder
来在数据处理和传输的时候进行序列化。 尽管encoder与标准的序列化都是将object 转为字节， 但是encoder是编码动态生成以及使用的， 允许spark执行很多操作如过滤，排序，以及hash， 并且不允许反序列化为一个object。

	// Encoders for most common types are automatically provided by importing sqlContext.implicits._
	val ds = Seq(1, 2, 3).toDS()
	ds.map(_ + 1).collect() // Returns: Array(2, 3, 4)

	// Encoders are also created for case classes.
	case class Person(name: String, age: Long)
	val ds = Seq(Person("Andy", 32)).toDS()

	// DataFrames can be converted to a Dataset by providing a class. Mapping will be done by name.
	val path = "examples/src/main/resources/people.json"
	val people = sqlContext.read.json(path).as[Person]

*** DataFrame通过指定一个类来转化为DataSet ***

### Interoperating with RDDs
spark sql 支持俩种方式将RDD转为DataFrames。 第一种方式是使用反射根据object中包含的类型来推断schema。 这种方式适用于你已经清楚的知道schema的情况下， 代码更简洁，更有效。

第二种方式是通过编程接口来构造一个schema，作用于已经存在的RDD上。 这种方式比较复杂，适用于列与类型都不清楚的情况下构造DataFrames。

##### Inferring the Schema Using Reflection
spark SQL提供的scala接口支持自动的将包含case class的RDD转为DataFrame。 case class定义了table的schema， case class的变量名被读取出来通过反射变为列明。 case classes可嵌套，或者包含复杂的数据类型，如Sequences或者Array。 RDD可以转为DataFrame，并注册成table。

	// sc is an existing SparkContext.
	val sqlContext = new org.apache.spark.sql.SQLContext(sc)
	// this is used to implicitly convert an RDD to a DataFrame.
	import sqlContext.implicits._

	// Define the schema using a case class.
	// Note: Case classes in Scala 2.10 can support only up to 22 fields. To work around this limit,
	// you can use custom classes that implement the Product interface.
	case class Person(name: String, age: Int)

	// Create an RDD of Person objects and register it as a table.
	val people = sc.textFile("examples/src/main/resources/people.txt").map(_.split(",")).map(p => Person(p(0), p(1).trim.toInt)).toDF()
	people.registerTempTable("people")

	// SQL statements can be run by using the sql methods provided by sqlContext.
	val teenagers = sqlContext.sql("SELECT name, age FROM people WHERE age >= 13 AND age <= 19")

	// The results of SQL queries are DataFrames and support all the normal RDD operations.
	// The columns of a row in the result can be accessed by field index:
	teenagers.map(t => "Name: " + t(0)).collect().foreach(println)

	// or by field name:
	teenagers.map(t => "Name: " + t.getAs[String]("name")).collect().foreach(println)

	// row.getValuesMap[T] retrieves multiple columns at once into a Map[String, T]
	teenagers.map(_.getValuesMap[Any](List("name", "age"))).collect().foreach(println)
	// Map("name" -> "Justin", "age" -> 19)

##### Programmatically Specifying the Schema
当case class事先无法被定义的时候(类如： 整条记录的结构被编码为一个字符串，或者一个文本数据集被转码出来，不同的用户使用不同的字段的时候)， 可以通过以下三部，创建DataFrame
1. 有原始的RDD创建行RDD
2. 根据第一步创建的RDD结构创建schema 结构类型 StructType
3. 通过sqlContext的createDataFrame方式 将schema 应用于行的RDD上

	// sc is an existing SparkContext.
	val sqlContext = new org.apache.spark.sql.SQLContext(sc)

	// Create an RDD
	val people = sc.textFile("examples/src/main/resources/people.txt")

	// The schema is encoded in a string
	val schemaString = "name age"

	// Import Row.
	import org.apache.spark.sql.Row;

	// Import Spark SQL data types
	import org.apache.spark.sql.types.{StructType,StructField,StringType};

	// Generate the schema based on the string of schema
	val schema =
	  StructType(
	    schemaString.split(" ").map(fieldName => StructField(fieldName, StringType, true)))

	// Convert records of the RDD (people) to Rows.
	val rowRDD = people.map(_.split(",")).map(p => Row(p(0), p(1).trim))

	// Apply the schema to the RDD.
	val peopleDataFrame = sqlContext.createDataFrame(rowRDD, schema)

	// Register the DataFrames as a table.
	peopleDataFrame.registerTempTable("people")

	// SQL statements can be run by using the sql methods provided by sqlContext.
	val results = sqlContext.sql("SELECT name FROM people")

	// The results of SQL queries are DataFrames and support all the normal RDD operations.
	// The columns of a row in the result can be accessed by field index or by field name.
	results.map(t => "Name: " + t(0)).collect().foreach(println)

# Data Sources
spark SQL 支持对 DataFrame接口加载的各种数据源进行操作。DataFrame 可以被当做一个普通的RDD来操作，也可以注册成为一个临时的表通过sql来访问数据。

### Generic Load/Save Functions

	val df = sqlContext.read.load("examples/src/main/resources/users.parquet")
	df.select("name", "favorite_color").write.save("namesAndFavColors.parquet")

### Manually Specifying Options
通过一些额外的参数可以手动的指定数据源， 数据源必须比如使用全面(如: org.apache.spark.sql.parquet), 但是内建的数据源可以使用简称(如: json, parquet, jdbc), 任何类型的DataFrame都可以通过这种方式转为令一种类型

	val df = sqlContext.read.format("json").load("examples/src/main/resources/people.json")
	df.select("name", "age").write.format("parquet").save("namesAndAges.parquet")

### Run SQL on files directly

	val df = sqlContext.sql("SELECT * FROM parquet.`examples/src/main/resources/users.parquet`")

### Save Modes

	在保存数据的时候，可以设置saveMode， 用来指明如何处理现有的数据。
	+ SaveMode.ErrorIfExists (default)	  "error"   当向数据源保存一个DataFrame时，如果数据已经存在， 就是抛出异常
	+ SaveMode.Append 					  "append"  当向数据源保存一个DataFrame时，如果数据已经存在， 则会追加到现有数据之后
	+ SaveMode.Overwrite 				  "overwrite"  当向数据源保存一个DataFrame时，如果数据已经存在， 则会覆盖现有的数据
	+ SaveMode.Ignore 					  "ignore"	   当向数据源保存一个DataFrame时，如果数据已经存在， 则不执行相应的操作

### Saving to Persistent Tables

使用HiveContext时， 可以将DataFrame通过saveAsTable命令存储在持久化表中。 不同于registerTempTable， saveAsTable将会保存DataFrame的内容数据，并在HiveMetastore中创建一个指向数据的指针。 当spark程序重新启动后， 只要连接之前的metastore， 持久化表会一直存在。 通过sqlContext的table方法可以重新从持久化表数据中创建table。saveAsTable 默认创建一张可管理的表，也就是说，表数据的位置由metastore来管理。 当表删除后，metadata的数据也会自动删除。

# Parquet Files
Parquet 是一种列存储数据结构， spark SQL对其提供了读写功能， 文件的schema保存在原始数据中。 当写Parquet文件时，所有的列都自动转为可为空的压缩格式

### Loading Data Programmatically

	import sqlContext.implicits._

	val people: RDD[Person] = ... // An RDD of case class objects, from the previous example.

	// The RDD is implicitly converted to a DataFrame by implicits, allowing it to be stored using Parquet.
	people.write.parquet("people.parquet")

	// Read in the parquet file created above. Parquet files are self-describing so the schema is preserved.
	// The result of loading a Parquet file is also a DataFrame.
	val parquetFile = sqlContext.read.parquet("people.parquet")

	//Parquet files can also be registered as tables and then used in SQL statements.
	parquetFile.registerTempTable("parquetFile")
	val teenagers = sqlContext.sql("SELECT name FROM parquetFile WHERE age >= 13 AND age <= 19")
	teenagers.map(t => "Name: " + t(0)).collect().foreach(println)

### Partition Discovery
表分区是一种通用的优化方式， 在分区表中，数据通常被存储在不同的目录中， 分区字段的值就是目录名。 Parquet数据源能够自动发现和推倒出分区信息。 例如： 我们以如下目录结构保存数据，gender和country为分区字段。

	path
	└── to
	    └── table
	        ├── gender=male
	        │   ├── ...
	        │   │
	        │   ├── country=US
	        │   │   └── data.parquet
	        │   ├── country=CN
	        │   │   └── data.parquet
	        │   └── ...
	        └── gender=female
	            ├── ...
	            │
	            ├── country=US
	            │   └── data.parquet
	            ├── country=CN
	            │   └── data.parquet
	            └── ...

将路径参数```  path/to/table ``` 传递给 ``` SQLContext.read.parquet```或者 ``` SQLContext.read.load ```, SparkSQL会自动的从路径上扩展出分区信息。 DataFrame的schema如下：

	root
	|-- name: string (nullable = true)
	|-- age: long (nullable = true)
	|-- gender: string (nullable = true)
	|-- country: string (nullable = true)

分区字段的类型也是被自动推倒出来的。 字符串以及数值类型的分区字段都支持。 有时，用户并不希望自动推断分区字段类型， 可以通过 ``` spark.sql.sources.partitionColumnTypeInference.enabled```来设置， 默认值为true。 当设置为false后，分区字段为string类型

spark1.6开始，默认的spark只会在给定路径下寻找分区字段。如用户使用path/to/table/gender=male 路径给 ``` SQLContext.read.parquet``` or ```SQLContext.read.load```, gender并不会作为分区字段。 如果你需要指定根路径从哪儿开始， 可以在datasource 中使用basePath参数。 类如，当数据路径为``` path/to/table/gender=male```, 且用户设置了basePath 为``` path/to/table/```. gender还是分区字段


### Schema Merging

与ProtocolBuffer, Avro, and Thrift类似， Parquet也支持schema演化。 用户可以先设置一个简单的schema， 然后逐渐的添加更多需要的字段。 慢慢的，用户会发现有很多的Parquet files，且schema不相同。 Parquet数据源会自动检测到这些年情况，并对这些文件schema进行合并

因为合并schema是一个比较费时的操作，所以从1.5.0开始，默认关闭了这个选项。 在下面情况下，你可以选择开启
1. 当读取Parquet fields
2. 在全局变量```spark.sql.parquet.mergeSchema```被设置为true时

	// sqlContext from the previous example is used in this example.
	// This is used to implicitly convert an RDD to a DataFrame.
	import sqlContext.implicits._

	// Create a simple DataFrame, stored into a partition directory
	val df1 = sc.makeRDD(1 to 5).map(i => (i, i * 2)).toDF("single", "double")
	df1.write.parquet("data/test_table/key=1")

	// Create another DataFrame in a new partition directory,
	// adding a new column and dropping an existing column
	val df2 = sc.makeRDD(6 to 10).map(i => (i, i * 3)).toDF("single", "triple")
	df2.write.parquet("data/test_table/key=2")

	// Read the partitioned table
	val df3 = sqlContext.read.option("mergeSchema", "true").parquet("data/test_table")
	df3.printSchema()

	// The final schema consists of all 3 columns in the Parquet files together
	// with the partitioning column appeared in the partition directory paths.
	// root
	// |-- single: int (nullable = true)
	// |-- double: int (nullable = true)
	// |-- triple: int (nullable = true)
	// |-- key : int (nullable = true)

### Hive metastore Parquet table conversion
在读取或者写 Hive metastore Parquet tables时， spark SQL 为了更好的性能， 使用自己的parquet，放弃hive的SerDe。 可以通过```spark.sql.hive.convertMetastoreParquet```来控制，默认为打开。

##### Hive/Parquet Schema Reconciliation

##### Metadata Refreshing

	sqlContext.refreshTable("my_table")

##### Configuration

	+ spark.sql.parquet.binaryAsString
	+ spark.sql.parquet.int96AsTimestamp
	+ spark.sql.parquet.cacheMetadata
	+ spark.sql.parquet.compression.codec
	+ spark.sql.parquet.filterPushdown
	+ spark.sql.hive.convertMetastoreParquet
	+ spark.sql.parquet.output.committer.class
	+ spark.sql.parquet.mergeSchema

### JSON Datasets
通过SQLContext.read.json() 加载一个字符串或者json文件，来生成一个DataFrame

	// sc is an existing SparkContext.
	val sqlContext = new org.apache.spark.sql.SQLContext(sc)

	// A JSON dataset is pointed to by path.
	// The path can be either a single text file or a directory storing text files.
	val path = "examples/src/main/resources/people.json"
	val people = sqlContext.read.json(path)

	// The inferred schema can be visualized using the printSchema() method.
	people.printSchema()
	// root
	//  |-- age: integer (nullable = true)
	//  |-- name: string (nullable = true)

	// Register this DataFrame as a table.
	people.registerTempTable("people")

	// SQL statements can be run by using the sql methods provided by sqlContext.
	val teenagers = sqlContext.sql("SELECT name FROM people WHERE age >= 13 AND age <= 19")

	// Alternatively, a DataFrame can be created for a JSON dataset represented by
	// an RDD[String] storing one JSON object per string.
	val anotherPeopleRDD = sc.parallelize(
	  """{"name":"Yin","address":{"city":"Columbus","state":"Ohio"}}""" :: Nil)
	val anotherPeople = sqlContext.read.json(anotherPeopleRDD)

### Hive Tables
构造HiveContext， 继承自SQLContext

	// sc is an existing SparkContext.
	val sqlContext = new org.apache.spark.sql.hive.HiveContext(sc)

	sqlContext.sql("CREATE TABLE IF NOT EXISTS src (key INT, value STRING)")
	sqlContext.sql("LOAD DATA LOCAL INPATH 'examples/src/main/resources/kv1.txt' INTO TABLE src")

	// Queries are expressed in HiveQL
	sqlContext.sql("FROM src SELECT key, value").collect().foreach(println)

##### Interacting with Different Versions of Hive Metastore

	+ spark.sql.hive.metastore.version
	+ spark.sql.hive.metastore.jars
	+ spark.sql.hive.metastore.sharedPrefixes
	+ spark.sql.hive.metastore.barrierPrefixes

### JDBC To Other Databases
spark SQL 也可以通过jdbc从其他的数据源中读取数据， 通过```JdbcRDD```来对数据进行操作。 返回DataFrame的结果，可以很容易在SparkSQL中处理，并且可以很容易的与其他数据源进行join操作。 JDBC datasource 也可以很容易的用java或者python来实现。

使用JDBC 需要现在spark的classpath中放入数据库对应的jdbc driver， 如，连接postgresql数据库，需要执行如下命令：

	SPARK_CLASSPATH=postgresql-9.3-1102-jdbc41.jar
	bin/spark-shell

参数

| 参数名 |  意义     |
|-------|-----------|
|url				|   连接的jdbc url       |
|dbtable				|      要读取的jdbc 数据表    |
|driver				|        连接url时需要的jdbc driver名称  |
|partitionColumn,lowerBound,upperBound,numPartitions				|   These options must all be specified if any of them is specified. They describe how to partition the table when reading in parallel from multiple workers. partitionColumn must be a numeric column from the table in question. Notice that lowerBound and upperBound are just used to decide the partition stride, not for filtering the rows in table. So all rows in the table will be partitioned and returned.       |
|fetchSize				| 决定了每一轮查询返回的行数         |



	val jdbcDF = sqlContext.read.format("jdbc").options(Map("url" -> "jdbc:postgresql:dbserver", "dbtable" -> "schema.tablename")).load()


