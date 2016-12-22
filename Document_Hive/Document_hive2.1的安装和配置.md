title: hive2.1 的安装和配置
notebook: 技术相关
tags: hive

[TOC]

目前hive的最新版本是2.1.0， 你可以下载tar包安装，也可以通过源码编译安装

## 安装条件
+ java1.7
+ hadoop 2.*
+ hive 可以运行在linux和windows环境下。 mac一般用于开发环境.

## 安装hive稳定版本

从apache hive官方下载稳定版的hive， 并且解压

	 $ tar -xzvf hive-x.y.z.tar.gz

将HIVE_HOME变量指定到hive的安装目录下

	$ cd hive-x.y.z
  	$ export HIVE_HOME={{pwd}}

将HIVE_HOME加入到环境变量

	$ export PATH=$HIVE_HOME/bin:$PATH

## 从源码编译安装hive

从git上clone源码，位置 [https://git-wip-us.apache.org/repos/asf/hive.git](https://git-wip-us.apache.org/repos/asf/hive.git)； 目前hive是通过[maven](http://maven.apache.org/) 来编译的


#### 编译master分支

	$ git clone https://git-wip-us.apache.org/repos/asf/hive.git
	$ cd hive
	$ mvn clean package -Pdist
	$ cd packaging/target/apache-hive-{version}-SNAPSHOT-bin/apache-hive-{version}-SNAPSHOT-bin
	$ ls
	LICENSE
	NOTICE
	README.txt
	RELEASE_NOTES.txt
	bin/ (all the shell scripts)
	lib/ (required jar files)
	conf/ (configuration files)
	examples/ (sample input and query files)
	hcatalog / (hcatalog installation)
	scripts / (upgrade scripts for hive-metastore)

#### 编译branch-1分支
branch-1 的hive支持Hadoop 1.x 和 2.x， 通过maven profile 来区分hadoop的版本， 编译hadoop 1.x 的版本使用hadoop-1, 编译hadoop 2.x 的版本使用 hadoop-2 , 如编译hadoop 1.x的时候命令如下：

	$ mvn clean package -Phadoop-1,dist

#### 编译0.13之前的版本
编译0.13之前的版本，需要使用apache ant, 支持hadoop0.20

	$ svn co http://svn.apache.org/repos/asf/hive/branches/branch-{version} hive
	$ cd hive
	$ ant clean package
	$ cd build/dist
	# ls
	LICENSE
	NOTICE
	README.txt
	RELEASE_NOTES.txt
	bin/ (all the shell scripts)
	lib/ (required jar files)
	conf/ (configuration files)
	examples/ (sample input and query files)
	hcatalog / (hcatalog installation)
	scripts / (upgrade scripts for hive-metastore)

编译0.13之前的版本，需要使用apache ant, 支持hadoop0.23，2.0.0 或者其他版本

	$ ant clean package -Dhadoop.version=0.23.3 -Dhadoop-0.23.version=0.23.3 -Dhadoop.mr.rev=23
  	$ ant clean package -Dhadoop.version=2.0.0-alpha -Dhadoop-0.23.version=2.0.0-alpha -Dhadoop.mr.rev=23

## 运行
hive需要hadoop，所以
+ 将hadoop添加到你的路径下
+ 或者设置环境变量 ```export HADOOP_HOME=<hadoop-install-dir>```

通过hdfs 相关命令创建```/tmp``` 和  ```/user/hive/warehouse(aka hive.metastore.warehouse.dir)``` 并在创建表之前，给俩个目录授权 g + w

	$ $HADOOP_HOME/bin/hadoop fs -mkdir       /tmp
  	$ $HADOOP_HOME/bin/hadoop fs -mkdir       /user/hive/warehouse
  	$ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /tmp
  	$ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /user/hive/warehouse

设置HIVE_HOME环境变量，这个虽然不是必须的，但是比较有用

	$ export HIVE_HOME=<hive-install-dir>

### 运行hive 客户端
运行hive 命令行

	 $ $HIVE_HOME/bin/hive

### 运行[hiveserver2](http://hiveserver2) 和 Beeline
从hive2.1开始， 运行时需要通过schematool命令初始化。 类如，要使用derby作为数据源

	$ $HIVE_HOME/bin/schematool -dbType <db type> -initSchema

hiveserver2 有它自己的命令行叫beeline。 hive命令行已经不建议使用了, beeline开始被更多人使用。 beeline支持多用户， 安全，且hiveserver2 包含了更多的功能。 运行hiveserver2 和beeline

	$ $HIVE_HOME/bin/hiveserver2
  	$ $HIVE_HOME/bin/beeline -u jdbc:hive2://$HS2_HOST:$HS2_PORT

beeline 以hiveserver2的url(默认为默认为```localhost:10000```)开头， 形如``` jdbc:hive2://localhost:10000```
测试时可以将hiveserver2与beeline在同一进程中启动， 类如

	$ $HIVE_HOME/bin/beeline -u jdbc:hive2://

**需要先启动hiveserver2， 才能通过beeline连接**

#### 出现的相关问题

1. 通过beeline连接hiveserver2 时，报错 ```hadoop is not allowed to impersonate anonymou```

	解决方案：
	+ 修改```core-site.xml```文件

		```
		  <property>
		    <name>hadoop.proxyuser.hadoop.hosts</name>
		    <value>*</value>
		  </property>
		  <property>
		    <name>hadoop.proxyuser.hadoop.groups</name>
		    <value>*</value>
		  </property>
		```

	+ 重启hdfs

2. ***通过beeline 连接hiveserver， 创建hbase外部表时，无法连接到hbase启动的一个节点， 而通过hive cli 执行时没有问题的***

	在确保各节点联通性的情况下，如果还出现上述问题， 很有可能是版本不匹配造成的，及hive与hbase的版本兼容问题。 为了不再版本上造成困扰，尽量选CDH统一的包;


### 运行HCatalog
hive0.11.0以及之后的版本启动HCatalog 如下：

	$ $HIVE_HOME/hcatalog/sbin/hcat_server.sh

hive0.11.0之前的版本启动HCatalog如下：

	 $ $HIVE_HOME/hcatalog/bin/hcat

启动hcatalog_web

	 $ $HIVE_HOME/hcatalog/sbin/webhcat_server.sh

### hive 配置管理概览

+ hive默认从``` <install-dir>/conf/hive-default.xml ``` 获取配置信息
+ 通过环境变量 ```HIVE_CONF_DIR``` 可以修改hive默认配置文件的路径
+ 通过在``` <install-dir>/conf/hive-site.xml ``` 配置来改变默认配置
+ hive log4j 的配置被存储在 ```  <install-dir>/conf/hive-log4j.properties ```
+ hive的配置信息是覆盖在hadoop配置之上的，也就是说，hadoop的配置先与hive被加载
+ 变更hive配置信息的方式有
	* 修改 ``` hive-site.xml ```, 配置你想要的参数
	* 使用 set 命令
	* 使用如下的语法调用hive，beeline和hiveserver2
		- ```$ bin/hive --hiveconf x1=y1 --hiveconf x2=y2  //this sets the variables x1 and x2 to y1 and y2 respectively```
		- ```$ bin/hiveserver2 --hiveconf x1=y1 --hiveconf x2=y2  //this sets server-side variables x1 and x2 to y1 and y2 respectively```
		- ```$ bin/beeline --hiveconf x1=y1 --hiveconf x2=y2  //this sets client-side variables x1 and x2 to y1 and y2 respectively.```
	* 设置HIVE_OPTS 环境变量， 如 ```"--hiveconf x1=y1 --hiveconf x2=y2```

### 运行时配置
+  hive查询是通过map-reduce 执行的， 所以hive查询也受到hadoop配置的影响
+  hive命令行和beeline 中的set 可以设置任意的hadoop或者hive的配置参数 如

	beeline> SET mapred.job.tracker=myhost.mycompany.com:50030;
    beeline> SET -v;

## Hive, Map-Reduce and Local-Mode
hive 大多数的查询都被转化为的mr的job， 然后提交到mr集群来执行， 参数为：

	mapred.job.tracker

该参数用于指向多个节点的mr集群。hadoop 也提供了一个比较优雅的选项，在用户自己的工作空间运行mr job， 对于小的数据集，这种做法是非常有用的， 因为本地模式执行比提交job到大集群上要快的多。但是，本地模式只能运行一个reduce, 而且在处理大数据集的时候会比较慢。

从release0.7开始，hive全面支持本地模式运行， 可以通过如下命令来启动本地模式

	 hive> SET mapreduce.framework.name=local;

默认本地模式不启动的， 在启动本地模式后，hive会分析查询中每个mr的size， 当满足以下条件时，进入本地模式执行
+ job 总的输入小于 ```hive.exec.mode.local.auto.inputbytes.max (128MB by default) ```
+ map 的数量小于 ``` hive.exec.mode.local.auto.input.files.max/hive.exec.mode.local.auto.tasks.max (4 by default) ```
+ reduce 的数据为 1或0

因此，基于小数据集的查询，或者在大的数据集上执行查询，但是输入数据量的子查询获取的数据量较小的情况下，hive job会在本地模式下执行。
这个时候，在hadoop server上运行和在hive client运行就会出现不同(因为hadoop server和hive client 的jvm 或者包不一致)， 本地模式下会出现无法预期的错误，同时，本地模式的执行时在hive client的一个子进程中运行的，所以需要通过```hive.mapred.local.mem```来设置子进程的最大内存数。默认值为0， 由hadoop来决定子进程的内存大小。

## hive log
hive 通过log4j来记录日志， 0.13.0之前的版本，日志等级为WARN ，从0.13.0开始，日志等级为INFO

log默认存储的目录为``` /tmp/<user.name>:```
+ ```/tmp/<user.name>/hive.log```

更改log目录，设置``` $HIVE_HOME/conf/hive-log4j.properties ``` 中的hive.log.dir

	hive.log.dir=<other_location>

hive log默认不会打印在控制台，如果需要打印在控制台，需要启动客户端的时候，添加参数

	bin/hive --hiveconf hive.root.logger=INFO,console  //for HiveCLI (deprecated)
	bin/hiveserver2 --hiveconf hive.root.logger=INFO,console

hive log的日志等级也可以变更

	bin/hive --hiveconf hive.root.logger=INFO,DRFA //for HiveCLI (deprecated)
	bin/hiveserver2 --hiveconf hive.root.logger=INFO,DRFA

另一个log的配置为TimeBasedRollingPolicy ， 可以设置日志按日志滚动存档

	bin/hive --hiveconf hive.root.logger=INFO,DAILY //for HiveCLI (deprecated)
	bin/hiveserver2 --hiveconf hive.root.logger=INFO,DAILY

**hive.root.logger 并不能通过set来设置，因为log的参数是在初始化的时候获取的**

hive 默认的把每个查询session的日志保存在``` /tmp/<user.name>/``` 的目录下， 可以通过设置hive-site.xml 中的 ```hive.querylog.location```变量来更改这个日志的目录

hive在hadoop集群上执行查询的日志，是由hadoop 配置来控制的。 通常，hadoop会在每个map和reduce执行完成后产生一个日志文件，保存在hdfs上。 可以通过job tracker的web UI
来获取这些日志信息。

当我们采用本地模式时(mapreduce.framework.name=local), hadoop/hive的执行日志都会保存在客户端本地。 从hive release0.6开始， 使用hive-exec-log4j.properties来决定日志的分发， 默认会产生一个日志文件，保存在```/tmp/<user.name>```目录下，

### Audit Logs
Audit Logs 用于记录每个metastore api的调用， 日志名为```HiveMetaStore.audit```, 使用log4j来记录日志，地址等级为INFO

### Perf Logger
通过PerfLogger来获取性能数据， 日志等级为DEBUG， 通过设置log4j 来获取性能日志

	log4j.logger.org.apache.hadoop.hive.ql.log.PerfLogger=DEBUG


## DDL Operations

### 创建表

	 hive> CREATE TABLE pokes (foo INT, bar STRING);

创建pokes表，包含俩列 foo和bar

	 hive> CREATE TABLE invites (foo INT, bar STRING) PARTITIONED BY (ds STRING);

创建表invites表，包含分区字段ds

### 浏览表

	hive> SHOW TABLES;

显示所有的表

	hive> SHOW TABLES '.*s';

显示所有以s结尾的表

	hive> DESCRIBE invites;

展现表结果

### 更改或者删除表

	hive> ALTER TABLE events RENAME TO 3koobecaf;
  	hive> ALTER TABLE pokes ADD COLUMNS (new_col INT);
  	hive> ALTER TABLE invites ADD COLUMNS (new_col2 INT COMMENT 'a comment');
  	hive> ALTER TABLE invites **REPLACE COLUMNS** (foo INT, bar STRING, baz INT COMMENT 'baz replaces new_col2');

REPLACE COLUMNS 替换表中存在的所有列，也可以用于从schema删除列

	hive> ALTER TABLE invites REPLACE COLUMNS (foo INT COMMENT 'only keep the first column');

删除表

	 hive> DROP TABLE pokes;

### Metadata Store
metadata默认是一个内嵌的derby数据库，数据存储于本地磁盘, 可以通过```javax.jdo.option.ConnectionURL```来配置，默认数据存储在```./metastore_db```下。

metadata 可以存储于任何支持JPOX的数据库。 RDBMS类型可以通过```javax.jdo.option.ConnectionURL```和```javax.jdo.option.ConnectionDriverName```来设置。  JDO文档含有更多数据库细节。 数据库schema 在 ```src/contrib/hive/metastore/src/model```下的.jdo文件中申明。


## DML Operations

load数据到hive

	 hive> LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;

加载一个文件到pokes表，文件中含有俩列，通过ctrl-a分割。 'LOCAL' 表示了输入的文件是本地的。 去掉‘LOCAL’， 则表明文件来源于HDFS。 'overwrite' 说明表中的数据会被删除， 如果去掉'overwrite'， 数据会被追加到原有数据后。

注：

	+ load命令会根据schema 对数据做预处理
	+ 如果load的文件在hdfs， 则会被移到hive控制的文件系统namespace下。

hive的文件根目录可以在hive-site.xml中的```hive.metastore.warehouse.dir```来指定。 建议用户在通过hive创建表之前，预先创建目录。

	hive> LOAD DATA LOCAL INPATH './examples/files/kv2.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-15');
  	hive> LOAD DATA LOCAL INPATH './examples/files/kv3.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-08');

数据被load到invites表的俩个不同的分区里， 所以invites表在创建的时候就必须创建分区字段。

	hive> LOAD DATA INPATH '/user/myname/kv2.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-15');

上述命令将数据从HDFS 目录load到表中。

## SQL Operations

一些简单的查询会被展示。更多的例子请查看 ```build/dist/examples/queries```, 以及```ql/src/test/queries/positive```

### SELECTS and FILTERS

	 hive> SELECT a.foo FROM invites a WHERE a.ds='2008-08-15';

从invites表的'2008-08-15'分区中查询 'foot'字段。 结果不会被保存，而是直接显示在控制台。

	 hive> INSERT OVERWRITE DIRECTORY '/tmp/hdfs_out' SELECT a.* FROM invites a WHERE a.ds='2008-08-15';

将查询结果覆盖写入到hdfs的/tmp/hdfs_out 目录下。

	 hive> INSERT OVERWRITE LOCAL DIRECTORY '/tmp/local_out' SELECT a.* FROM pokes a;

将查询结果写到本地

	hive> INSERT OVERWRITE TABLE events SELECT a.* FROM profiles a;
  	hive> INSERT OVERWRITE TABLE events SELECT a.* FROM profiles a WHERE a.key < 100;
  	hive> INSERT OVERWRITE LOCAL DIRECTORY '/tmp/reg_3' SELECT a.* FROM events a;
  	hive> INSERT OVERWRITE DIRECTORY '/tmp/reg_4' select a.invites, a.pokes FROM profiles a;
  	hive> INSERT OVERWRITE DIRECTORY '/tmp/reg_5' SELECT COUNT(*) FROM invites a WHERE a.ds='2008-08-15';
  	hive> INSERT OVERWRITE DIRECTORY '/tmp/reg_5' SELECT a.foo, a.bar FROM invites a;
  	hive> INSERT OVERWRITE LOCAL DIRECTORY '/tmp/sum' SELECT SUM(a.pc) FROM pc1 a;

** insert 可以将结果写到table, local directory 和 hdfs directory**
可以对一列的sum, avg, max进行select。 一般的用count(1) 代替 count(*)

### group by

	hive> FROM invites a INSERT OVERWRITE TABLE events SELECT a.bar, count(*) WHERE a.foo > 0 GROUP BY a.bar;
 	hive> INSERT OVERWRITE TABLE events SELECT a.bar, count(*) FROM invites a WHERE a.foo > 0 GROUP BY a.bar;

### JOIN

	hive> FROM pokes t1 JOIN invites t2 ON (t1.bar = t2.bar) INSERT OVERWRITE TABLE events SELECT t1.bar, t1.foo, t2.foo;

### MULTITABLE INSERT

	FROM src
  	INSERT OVERWRITE TABLE dest1 SELECT src.* WHERE src.key < 100
  	INSERT OVERWRITE TABLE dest2 SELECT src.key, src.value WHERE src.key >= 100 and src.key < 200
  	INSERT OVERWRITE TABLE dest3 PARTITION(ds='2008-04-08', hr='12') SELECT src.key WHERE src.key >= 200 and src.key < 300
  	INSERT OVERWRITE LOCAL DIRECTORY '/tmp/dest4.out' SELECT src.value WHERE src.key >= 300;

### STREAMING

	hive> FROM invites a INSERT OVERWRITE TABLE events SELECT TRANSFORM(a.foo, a.bar) AS (oof, rab) USING '/bin/cat' WHERE a.ds > '2008-08-09';

通过/bin/cat将数据在map阶段流式的注入， 同样的，我们也可以在reduce阶段注入。
[Turorial](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-Custommap%2Freducescripts)


** hive表的默认分割符为ascii码的控制符 \001 **


## 实战
### 启动hiveserver2

	nohup bin/hive --service hiveserver2 --hiveconf hive.rooter.logger=hive.root.logger=INFO,console > hiveserver2.log &

### 创建带分区的hive extend table

创建分区表

	CREATE EXTERNAL TABLE visit_log(
		timestamp String COMMENT "时间",
		request String COMMENT "请求",
		path String COMMENT "日志路径",
		size BigInt COMMENT "数据大小",
		http_host String COMMENT "请求host",
		host String COMMENT "host",
		agent String COMMENT "客户端",
		backtime Double COMMENT "返回时间",
		clientip String COMMENT "客户端IP",
		status int COMMENT "状态",
		responsetime double COMMENT "响应时间",
		referer String COMMENT "referer",
		version String COMMENT "版本",
		xff String COMMENT "x-form-forward"
		)
	 partitioned by (date string)
	 ROW FORMAT DELIMITED FIELDS TERMINATED BY "\001"
	 LOCATION "/user/dw/tv";

创建分区

	ALTER TABLE visit_log ADD PARTITION (date='2016-10-17') LOCATION '2016-10-17';

**创建外部表的时候，location指明数据文件的目录，目录中包含数据文件**

### hive表metadata中文乱码的问题解决
