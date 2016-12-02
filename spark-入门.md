title: spark 编译
notebook: 技术相关
tags: spark

\[TOC\]

# 编译Apache Spark

## Apache Maven

基于maven构建apache spark。 需要maven3.3.9及以上和java7+

### 设置maven 内存

配置maven，设置MAVEN\_OPTS，🔟maven比正常情况获得更多的内存

```
export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=512m"
```

如果使用java7编译，还需要在MAVEN\_OPTS 中添加 配置 `-XX:MaxPermSize=512M， 否则就会出现如下的警告信息`

```
[INFO] Compiling 203 Scala sources and 9 Java sources to /Users/me/Development/spark/core/target/scala-2.11/classes...
[ERROR] PermGen space -> [Help 1]
[INFO] Compiling 203 Scala sources and 9 Java sources to /Users/me/Development/spark/core/target/scala-2.11/classes...
[ERROR] Java heap space -> [Help 1]
[INFO] Compiling 233 Scala sources and 41 Java sources to /Users/me/Development/spark/sql/core/target/scala-{site.SCALA_BINARY_VERSION}/classes...
OpenJDK 64-Bit Server VM warning: CodeCache is full. Compiler has been disabled.
OpenJDK 64-Bit Server VM warning: Try increasing the code cache size using -XX:ReservedCodeCacheSize=
```

Note:

* 在没有设置MAVEN\_OPTS情况下使用build/mvn, 脚本会自动的将上述选项添加到MAVEN\_OPTS添加到环境变量上
* test会自动的将这些选项添加到MAVEN\_OPT上，即便没有使用build/mvn
* 在使用java8 和build/mvn 编译的时候，会出现如下告警信息"ignoring option MaxPermSize=1g; support was removed in 8.0", 请忽略。

### build/mvn

spark现在自带了maven安装， 简化了spark的构建和发布，目录在build/ 下。 脚本会自动的下载和安装所有必须的依赖（maven, scala, 和 zinc）。 之前安装过的maven是可以使用的，然而，你需要适当的将scala和zinc的版本降低，以适应当前编译的版本需要。 build/mvn 通过以下参数控制，使得很容易去编译之前的一个版本，类如， 编译一个版本的spark如下：

```
./build/mvn -Pyarn -Phadoop-2.4 -Dhadoop.version=2.4.0 -DskipTests clean package
```

## Building a Runnable Distribution

创建一个和下载页下载的包相当的可运行的分支，可以通过根目录下的 `./dev/make-distribution.sh 来完成。 如：`

```
./dev/make-distribution.sh --name custom-spark --tgz -Psparkr -Phadoop-2.4 -Phive -Phive-thriftserver -Pyarn
```

更多信息可运行 `./dev/make-distribution.sh --help`

## Specifying the Hadoop Version

由于hdfs各版本协议不兼容， 如果你想从hdfs读取数据，需要根据你环境的hdfs版本来构建相应的spark。 这里可以通过参数`hadoop.version来指定hdfs版本，如果没有设置， 默认spark会针对hadoop2.2.0来构建。 版本的详细对应关系如下：`

| Hadoop version | Profile required |
| --- | --- |
| 2.2.x | hadoop-2.2 |
| 2.3.x | hadoop-2.3 |
| 2.4.x | hadoop-2.4 |
| 2.6.x | hadoop-2.6 |
| 2.7.x and later | hadoop-2.7 |如果你使用的yarn的版本与hadoop版本不同，可以设置`yarn.version 来指定yarn的版本， spark支持的yarn的版本为2.2.0或者以上`

```
# Apache Hadoop 2.2.X
./build/mvn -Pyarn -Phadoop-2.2 -DskipTests clean package

# Apache Hadoop 2.3.X
./build/mvn -Pyarn -Phadoop-2.3 -Dhadoop.version=2.3.0 -DskipTests clean package

# Apache Hadoop 2.4.X or 2.5.X
./build/mvn -Pyarn -Phadoop-2.4 -Dhadoop.version=VERSION -DskipTests clean package

# Apache Hadoop 2.6.X
./build/mvn -Pyarn -Phadoop-2.6 -Dhadoop.version=2.6.0 -DskipTests clean package

# Apache Hadoop 2.7.X and later
./build/mvn -Pyarn -Phadoop-2.7 -Dhadoop.version=VERSION -DskipTests clean package

# Different versions of HDFS and YARN.
./build/mvn -Pyarn -Phadoop-2.3 -Dhadoop.version=2.3.0 -Dyarn.version=2.2.0 -DskipTests clean package
```

## Building With Hive and JDBC Support

为了能够使用hive的jdbc server和控制台来调用spark sql， 构建的时候需要在原有参数的基础上添加 `-Phive 和 Phive-thriftserver 俩个属性。 默认spark支持hive1.2.1`

```
# Apache Hadoop 2.4.X with Hive 1.2.1 support
./build/mvn -Pyarn -Phadoop-2.4 -Dhadoop.version=2.4.0 -Phive -Phive-thriftserver -DskipTests clean package
```

## Packaging without Hadoop Dependencies for YARN

通过`mvn package构建的文件夹，默认包含了spark所有的依赖，包括hadoop以及其生态系统的工程。 这样就会出现，在yarn发布的时候，在excutor的环境变量中，会出现不同版本的依赖： spark构建生成的版本和每个节点在yarn.application.classpath 目录下包含的版本。 hadoop-provided属性在构建的时候，没有包含hadoop生态系统的工程，如zk,hadoop等。`

## Building for Scala 2.10

使用-Dscala-2.10属性

```
./dev/change-scala-version.sh 2.10
./build/mvn -Pyarn -Phadoop-2.4 -Dscala-2.10 -DskipTests clean package
```

## 单独编译子模块

通过mvn -pl 选项可以单独编译spark子模块

```
./build/mvn -pl :spark-streaming_2.11 clean install
```

## Continuous Compilation

`scala-maven-plugin maven插件支持持续构建`

```
./build/mvn scala:cc
```

以下几点需要注意：

* 持续编译cc 只监控src/main 和src/test 俩个目录，所以它只能运行在这种结构的子模块中
* 一般的需要在父目录中运行`mvn install， 子模块就可以正常工作，这是因为子模块是通过 spark-parent 模块来依赖其他子模块的。`

因此，一个完整的持续编译的流程如下：

```
$ ./build/mvn install
$ cd core
$ ../build/mvn scala:cc
```

## Speeding up Compilation with Zinc

zinc 是sbt增量编译的长期运行版，当它在后台运行的时候，它可以加速像spark这样的基于scala开发的项目的编译速度。用maven经常的重新编译spark的开发者，一定会对zinc感兴趣的。

执行build/mvn ， zinc会被自动的下载并安装， zinc进程默认绑定3030端口，可以通过设置`ZINC_PORT来设置， zinc进程会在第一次build/mvn的时候被调起， 当执行 build/zinc-<version>/bin/zinc -shutdown 被关闭。`

## Building with SBT

maven 是官方推荐的构建打包spark的方式， 但是sbt也慢慢被支持起来，因为sbt能够提供不断的迭代编译，所以sbt的呼声越来越高
sbt的构建也是源于maven的pom文件，所以sbt构建的时候使用和maven同样的参数。类如：

```
./build/sbt -Pyarn -Phadoop-2.3 package
```

## Encrypted Filesystems

在一个已编码的文件系统上构建spark的时候，会报 `Filename too long的错误。 需要在pom.xml的scala-maven-plugin下，添加如下配置`

```
<arg>-Xmax-classfile-name</arg>
<arg>128</arg>
```

在project/SparkBuild.scala中添加

```
scalacOptions in Compile ++= Seq("-Xmax-classfile-name", "128"),
```



