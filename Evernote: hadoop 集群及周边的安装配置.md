---
title: hadoop 集群及周边的安装配置
tags: ["HBase", "hadoop", "hive", "hue"]
notebook: 技术相关
---

# 版本选择
+ flume-ng-1.6.0-cdh5.8.0.tar.gz
+ hadoop-2.6.0-cdh5.8.0.tar.gz
+ hbase-1.2.0-cdh5.8.0.tar.gz
+ hbase-solr-1.5-cdh5.8.0.tar.gz
+ hive-1.1.0-cdh5.8.0.tar.gz
+ hue-3.9.0-cdh5.8.0.tar.gz
+ oozie-4.1.0-cdh5.8.0.tar.gz
+ pig-0.12.0-cdh5.8.0.tar.gz
+ solr-4.10.3-cdh5.8.0.tar.gz
+ spark-1.6.0-cdh5.8.0.tar.gz
+ sqoop-1.4.6-cdh5.8.0.tar.gz
+ sqoop2-1.99.5-cdh5.8.0.tar.gz
+ zookeeper-3.4.5-cdh5.8.0.tar.gz

# 准备

	10.19.138.198   thadoop-uelrcx-host1					 namenode  			resourcemanager	 hmaster		hiveserver2		master
	10.19.134.88    thadoop-uelrcx-host2	zk journalnode   namenode/datanode  nodemanager		 regionserver					worker
	10.19.164.182   thadoop-uelrcx-host3	zk journalnode   datanode  			nodemanager		 regionserver					worker
	10.19.78.105    thadoop-uelrcx-host4	zk journalnode   datanode  			nodemanager		 regionserver					worker

#### 设置免密码登录

# zookeeper 集群安装配置

#### 修改zk配置文件

	mv zoo_sample.cfg zoo.cfg

#### 修改zk参数


	tickTime=2000
	initLimit=10
	syncLimit=5
	dataDir=/tmp/zookeeper
	clientPort=2181
	server.2=thadoop-uelrcx-host2:2888:3888
	server.3=thadoop-uelrcx-host3:2888:3888
	server.4=thadoop-uelrcx-host4:2888:3888

#### 拷贝zk到thadoop-uelrcx-host2，thadoop-uelrcx-host3，thadoop-uelrcx-host3三台机器上

#### 在三台机器的/tmp/zookeeper下创建myid, 并分别填充2,3,4

#### 分别启动zk实例

	bin/zkServer.sh start

#### 验证是否启动成功

	bin/zkServer.sh status

#### 连接zk服务

	bin/zkCli.sh -server *********

# hadoop 安装

#### 修改core-site.xml

	<configuration>
	  <property>
	    <name>fs.defaultFS</name>
	    <value>hdfs://thadoopcluster</value>
	  </property>
	  <property>
	    <name>io.file.buffer.size</name>
	    <value>131072</value>
	  </property>
	  <property>
	    <name>ha.zookeeper.quorum</name>
	    <value>thadoop-uelrcx-host2:2181,thadoop-uelrcx-host3:2181,thadoop-uelrcx-host4:2181</value>
	  </property>
	  <property>
	    <name>hadoop.tmp.dir</name>
	    <value>/data/hadoop/tmp</value>
	    <description>Abase for other temporary directories.</description>
	  </property>
	  <property>
	    <name>hadoop.proxyuser.hadoop.hosts</name>
	    <value>*</value>
	  </property>
	  <property>
	    <name>hadoop.proxyuser.hadoop.groups</name>
	    <value>*</value>
	  </property>
	</configuration>


#### 修改hdfs-site.xml

	<configuration>
	    <property>
	      <name>dfs.nameservices</name>
	      <value>thadoopcluster</value>
	    </property>
	    <property>
	      <name>dfs.ha.namenodes.thadoopcluster</name>
	      <value>nn1, nn2</value>
	    </property>

	    <property>
	      <name>dfs.namenode.rpc-address.thadoopcluster.nn1</name>
	      <value>thadoop-uelrcx-host1:9000</value>
	    </property>

	    <property>
	      <name>dfs.namenode.rpc-address.thadoopcluster.nn2</name>
	      <value>thadoop-uelrcx-host2:9000</value>
	    </property>

	    <property>
	      <name>dfs.namenode.http-address.thadoopcluster.nn1</name>
	      <value>thadoop-uelrcx-host1:50070</value>
	    </property>

	    <property>
	      <name>dfs.namenode.http-address.thadoopcluster.nn2</name>
	      <value>thadoop-uelrcx-host2:50070</value>
	    </property>

	    <property>
	      <name>dfs.namenode.shared.edits.dir</name>
	      <value>qjournal://thadoop-uelrcx-host2:8485;thadoop-uelrcx-host3:8485;thadoop-uelrcx-host4:8485/thadoopcluster</value>
	    </property>

	    <property>
	      <name>dfs.client.failover.proxy.provider.thadoopcluster</name>
	      <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	    </property>

	    <property>
	      <name>dfs.ha.fencing.methods</name>
	      <value>sshfence</value>
	    </property>

	    <property>
	      <name>dfs.ha.fencing.ssh.private-key-files</name>
	      <value>/home/hadoop/.ssh/id_rsa</value>
	    </property>

	     <property>
	      <name>dfs.journalnode.edits.dir</name>
	      <value>/data/hadoop/journal/node/local/data</value>
	    </property>

	    <property>
	      <name>dfs.ha.automatic-failover.enabled</name>
	      <value>true</value>
	    </property>

	    <property>
	        <name>dfs.namenode.name.dir</name>
	        <value>/data/hadoop/namenode</value>
	    </property>


	    <property>
	        <name>dfs.datanode.data.dir</name>
	        <value>/data/hadoop/datanode</value>
	    </property>

	    <property>
	        <name>dfs.replication</name>
	        <value>3</value>
	    </property>

	    <property>
	        <name>dfs.webhdfs.enabled</name>
	        <value>true</value>
	    </property>
	</configuration>

#### 修改mapred-site.xml

	<configuration>
	  <property>
	      <name>mapreduce.framework.name</name>
	      <value>yarn</value>
	  </property>
	</configuration>

#### 修改yarn-site.xml

	<configuration>
	<!-- Site specific YARN configuration properties -->
	    <property>
	       <name>yarn.resourcemanager.connect.retry-interval.ms</name>
	       <value>2000</value>
	    </property>
	    <property>
	       <name>yarn.resourcemanager.ha.enabled</name>
	       <value>true</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.ha.rm-ids</name>
	      <value>rm1,rm2</value>
	    </property>
	    <property>
	       <name>yarn.resourcemanager.ha.automatic-failover.enabled</name>
	       <value>true</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.hostname.rm1</name>
	      <value>thadoop-uelrcx-host1</value>
	    </property>
	    <property>
	       <name>yarn.resourcemanager.hostname.rm2</name>
	       <value>thadoop-uelrcx-host2</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.ha.id</name>
	      <value>rm1</value>
	      <description>If we want to launch more than one RM in single node, we need this configuration</description>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.recovery.enabled</name>
	      <value>true</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.zk-state-store.address</name>
	      <value>thadoop-uelrcx-host2:2181,thadoop-uelrcx-host3:2181,thadoop-uelrcx-host4:2181</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.store.class</name>
	      <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.zk-address</name>
	      <value>thadoop-uelrcx-host2:2181,thadoop-uelrcx-host3:2181,thadoop-uelrcx-host4:2181</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.cluster-id</name>
	      <value>thadoopcluster-yarn</value>
	    </property>
	    <property>
	      <name>yarn.app.mapreduce.am.scheduler.connection.wait.interval-ms</name>
	      <value>5000</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.address.rm1</name>
	      <value>thadoop-uelrcx-host1:8132</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.scheduler.address.rm1</name>
	      <value>thadoop-uelrcx-host1:8130</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.webapp.address.rm1</name>
	      <value>thadoop-uelrcx-host1:8188</value>
	    </property>
	    <property>
	       <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
	       <value>thadoop-uelrcx-host1:8131</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.admin.address.rm1</name>
	      <value>thadoop-uelrcx-host1:8033</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.ha.admin.address.rm1</name>
	      <value>thadoop-uelrcx-host1:23142</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.address.rm2</name>
	      <value>thadoop-uelrcx-host2:8132</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.scheduler.address.rm2</name>
	      <value>thadoop-uelrcx-host2:8130</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.webapp.address.rm2</name>
	      <value>thadoop-uelrcx-host2:8188</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.resource-tracker.address.rm2</name>
	      <value>thadoop-uelrcx-host2:8131</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.admin.address.rm2</name>
	      <value>thadoop-uelrcx-host2:8033</value>
	    </property>
	    <property>
	      <name>yarn.resourcemanager.ha.admin.address.rm2</name>
	      <value>thadoop-uelrcx-host2:23142</value>
	    </property>
	    <property>
	      <name>yarn.nodemanager.aux-services</name>
	      <value>mapreduce_shuffle</value>
	    </property>
	    <property>
	      <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
	      <value>org.apache.hadoop.mapred.ShuffleHandler</value>
	    </property>
	     <property>
	      <name>yarn.nodemanager.local-dirs</name>
	      <value>/data/hadoop/yarn/local</value>
	    </property>
	    <property>
	      <name>yarn.nodemanager.log-dirs</name>
	      <value>/data/hadoop/log</value>
	    </property>
	    <property>
	      <name>mapreduce.shuffle.port</name>
	      <value>23080</value>
	    </property>
	    <property>
	      <name>yarn.client.failover-proxy-provider</name>
	      <value>org.apache.hadoop.yarn.client.ConfiguredRMFailoverProxyProvider</value>
	    </property>
	    <property>
	        <name>yarn.resourcemanager.ha.automatic-failover.zk-base-path</name>
	        <value>/yarn-leader-election</value>
	        <description>Optional setting. The default value is /yarn-leader-election</description>
	    </property>
	</configuration>

#### 修改slaves

	thadoop-uelrcx-host2
	thadoop-uelrcx-host3
	thadoop-uelrcx-host4

#### 将hadoop安装文件分发到四台服务器上

#### 启动journalnode

在thadoop-uelrcx-host1执行

	sbin/hadoop-daemons.sh start journalnode

或者单独进入thadoop-uelrcx-host2,thadoop-uelrcx-host3,thadoop-uelrcx-host4 中分别执行

	sbin/hadoop-daemon.sh start journalnode

jps检查是否有journalnode 进程

#### 格式化HDFS

在thadoop-uelrcx-host1执行

	bin/hadoop namenode -format

启动namenode

	sbin/hadoop-daemon.sh start namenode

在thadoop-uelrcx-host2执行下面命令，完成准备节点同步信息

	bin/hdfs namenode -bootstrapStandby

#### 格式化ZK

	bin/hdfs zkfc -formatZK

#### 启动hdfs

在thadoop-uelrcx-host1执行下列命令， 启动dfs

	sbin/start-dfs.sh

#### 启动yarn

在thadoop-uelrcx-host1执行下列命令， 启动yarn

	sbin/start-yarn.sh

## HDFS 支持多地址网络

目前，很多情况下，hadoop都运行在多地址网络环境下，集群内部通过内网IP联通，集群外部则通过外网IP访问集群功能。 这样做有很多有点：
+ 安全： 集群内部通讯的网络和对外通讯的网络相隔离，保证数据的安全
+ 性能： 内网集群可以采用很高的网络带宽，如光纤，宽带，或者千兆网
+ Failover/Redundancy: 节点可以在多网络环境下应对网络的适配的失败

## 多网络地址环境下的hadoop配置修改

#### Ensuring HDFS Daemons Bind All Interfaces

默认情况下,hdfs 节点既可以使用hostname， 也可以使用IP。 无论哪种情况，hdfs 进程都只会绑定一个单独的ip，以保证其他网络无法访问。
在多网络地址环境下的解决方式，强制服务节点绑定IP网段 0.0.0.0, 不设置端口。

	<property>
	  <name>dfs.namenode.rpc-bind-host</name>
	  <value>0.0.0.0</value>
	  <description>
	    The actual address the RPC server will bind to. If this optional address is
	    set, it overrides only the hostname portion of dfs.namenode.rpc-address.
	    It can also be specified per name node or name service for HA/Federation.
	    This is useful for making the name node listen on all interfaces by
	    setting it to 0.0.0.0.
	  </description>
	</property>

	<property>
	  <name>dfs.namenode.servicerpc-bind-host</name>
	  <value>0.0.0.0</value>
	  <description>
	    The actual address the service RPC server will bind to. If this optional address is
	    set, it overrides only the hostname portion of dfs.namenode.servicerpc-address.
	    It can also be specified per name node or name service for HA/Federation.
	    This is useful for making the name node listen on all interfaces by
	    setting it to 0.0.0.0.
	  </description>
	</property>

	<property>
	  <name>dfs.namenode.http-bind-host</name>
	  <value>0.0.0.0</value>
	  <description>
	    The actual adress the HTTP server will bind to. If this optional address
	    is set, it overrides only the hostname portion of dfs.namenode.http-address.
	    It can also be specified per name node or name service for HA/Federation.
	    This is useful for making the name node HTTP server listen on all
	    interfaces by setting it to 0.0.0.0.
	  </description>
	</property>

	<property>
	  <name>dfs.namenode.https-bind-host</name>
	  <value>0.0.0.0</value>
	  <description>
	    The actual adress the HTTPS server will bind to. If this optional address
	    is set, it overrides only the hostname portion of dfs.namenode.https-address.
	    It can also be specified per name node or name service for HA/Federation.
	    This is useful for making the name node HTTPS server listen on all
	    interfaces by setting it to 0.0.0.0.
	  </description>
	</property>

#### Clients use Hostnames when connecting to DataNodes

默认情况下 HDFS 客户端通过namenode提供的IP地址来访问datanode， 然而这个ip有可能是客户端无法访问的。 解决方案就是通过datanode的hostname，经由DNS来访问datanode。

	<property>
	  <name>dfs.client.use.datanode.hostname</name>
	  <value>true</value>
	  <description>Whether clients should use datanode hostnames when
	    connecting to datanodes.
	  </description>
	</property>

#### DataNodes use HostNames when connecting to other DataNodes

特殊情况下， namanode无法通过ip来访问datanode， 此时可以配置hostname，由DNS来访问datanode

	<property>
	  <name>dfs.datanode.use.datanode.hostname</name>
	  <value>true</value>
	  <description>Whether datanodes should use datanode hostnames when
	    connecting to other datanodes for data transfer.
	  </description>
	</property>

#### Ensuring yarn Daemons Bind All Interfaces

	<property>
	  <name>yarn.nodemanager.bind-host</name>
	  <value>0.0.0.0</value>
	</property>

	<property>
	  <name>yarn-timeline-service.bind-host</name>
	  <value>0.0.0.0</value>
	</property>

	<property>
	  <name>yarn.resourcemanager.bind-host</name>
	  <value>0.0.0.0</value>
	</property>


# HBASE 安装配置

#### 禁用hbase自带的zk

修改conf/hbase-env.sh

	export HBASE_MANAGES_ZK=false

#### 修改hbase-site.xml

	<configuration>
	  <property>
	    <name>hbase.rootdir</name>
	    <value>hdfs://thadoop-uelrcx-host1:9000/hbase</value>
	  </property>
	  <property>
	    <name>hbase.cluster.distributed</name>
	    <value>true</value>
	  </property>
	  <property>
	    <name>hbase.zookeeper.quorum</name>
	    <value>thadoop-uelrcx-host2,thadoop-uelrcx-host3,thadoop-uelrcx-host4</value>
	  </property>
	  <property>
	    <name>hbase.zookeeper.property.dataDir</name>
	    <value>/data/hadoop/log/hbase/zookeeper</value>
	  </property>
	</configuration>

#### 修改regionservers

	thadoop-uelrcx-host2
	thadoop-uelrcx-host3
	thadoop-uelrcx-host4

#### 将hbase安装目录分发了四台机器上
在thadoop-uelrcx-host1启动hbase

	bin/start-hbase.sh

# hive 安装配置

#### 修改hive-env.sh，设置HADOOP_HOME

	HADOOP_HOME=/Users/junjie.cheng/Developers/hadoop-2.6.0-cdh5.8.0

#### 设置hive-site.xml

	<!-- 数据仓库connect -->
	<property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true&amp;autoReconnect=true&amp;characterEncoding=UTF-8</value>
       <description>the URL of the MySQL database</description>
    </property>

    <property>
      <name>hive.jobname.length</name>
      <value>30</value>
    </property>

    <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>hive</value>
     </property>

    <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>hive</value>
     </property>

    <property>
        <name>datanucleus.autoCreateSchema</name>
        <value>false</value>
    </property>
    <property>
      <name>datanucleus.fixedDatastore</name>
      <value>true</value>
    </property>

    <property>
       <name>datanucleus.autoStartMechanism</name>
       <value>SchemaTable</value>
    </property>

    <property>
      <name>hive.metastore.warehouse.dir</name>
      <value>/user/hive/warehouse</value>
     </property>
    <!--
    <property>
      <name>hive.metastore.uris</name>
      <value>thrift://thadoop-uelrcx-host1:9083,thrift://thadoop-uelrcx-host2:9083</value>
    </property>
    -->
    <property>
        <name>hive.support.concurrency</name>
        <value>true</value>
     </property>
    <property>
      <name>hive.zookeeper.quorum</name>
       <value>thadoop-uelrcx-host2,thadoop-uelrcx-host3,thadoop-uelrcx-host4</value>
    </property>
    <property>
      <name>hive.zookeeper.client.port</name>
       <value>2181</value>
    </property>

    <property>
      <name>hive.server2.thrift.port</name>
      <value>10000</value>
    </property>

    <property>
      <name>hive.aux.jars.path</name>
      <value>file:///data/hadoop/hive-1.1.0-cdh5.8.0/lib/hive-json-serde.jar,file:///data/hadoop/hive-1.1.0-cdh5.8.0/lib/hive-contrib.jar,file:///data/hadoop/hive-1.1.0-cdh5.8.0/lib/hive-serde.jar</value>
    </property>

    <!--hbase 融合-->
    <property>
      <name>hbase.zookeeper.quorum</name>
      <value>thadoop-uelrcx-host2,thadoop-uelrcx-host3,thadoop-uelrcx-host4</value>
    </property>

#### 添加必要的jar包

+ mysql-connector-java-3.1.14-bin.jar
+ hbase-client-1.2.0-cdh5.8.0.jar, hbase-common-1.2.0-cdh5.8.0.jar, hbase-hadoop-compat-1.2.0-cdh5.8.0.jar, hbase-hadoop2-compat-1.2.0-cdh5.8.0.jar
+ netty-all-4.0.23.Final.jar
+ metrics-core-2.2.0.jar

#### 启动hiveserver2

1. 通过schematool初始化数据源

	$HIVE_HOME/bin/schematool -dbType mysql -initSchema

2. 启动hiveserver2

	$HIVE_HOME/bin/hiveserver2

3. 通过beeline 连接hive

	$HIVE_HOME/bin/beeline -u jdbc:hive2://$HS2_HOST:$HS2_PORT

# spark 安装及配置

##  spark on yarn


#### 修改spark-env.sh, 设置

	export JAVA_HOME=/opt/jdk1.7.0_79
	export HADOOP_HOME=/data/hadoop/hadoop-2.6.0-cdh5.8.0
	export HADOOP_CONF_DIR=/data/hadoop/hadoop-2.6.0-cdh5.8.0/etc/hadoop
	export SPARK_CLASSPATH=$SPARK_CLASSPATH:/data/hadoop/hadoop-2.6.0-cdh5.8.0/share/hadoop/common/*:/data/hadoop/hadoop-2.6.0-cdh5.8.0/share/hadoop/common/lib/*:/data/hadoop/hadoop-2.6.0-cdh5.8.0/share/hadoop/yarn/*:/data/hadoop/hadoop-2.6.0-cdh5.8.0/share/hadoop/yarn/lib/*:/data/hadoop/spark-1.6.0-cdh5.8.0/lib/*:/data/hadoop/hive-1.1.0-cdh5.8.0/lib/*:/data/hadoop/hbase-1.2.0-cdh5.8.0/lib/*

#### 直接提交任务到yarn上执行

## spark  standalone

#### 修改slaves

	thadoop-uelrcx-host2
	thadoop-uelrcx-host3
	thadoop-uelrcx-host4

### 启动spark集群

	sbin/start-all.sh
