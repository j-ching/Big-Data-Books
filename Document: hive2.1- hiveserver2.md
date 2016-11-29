title: hive2.1- hiveserver2
notebook: 技术相关
tags: hive, hive2.1, hiveserver2

[TOC]

# HiveServer2
HS2 是一个服务接口，供客户端提交查询并且返回结果。它比HiveServer1更有优势， 虽然hiveserver1与hiveserver2都是通过thrift server提供的服务，但是hiveserver1不支持多个客户端同时请求。 而HS2提供了多客户端使用和授权，而且提供额更加友好的类似于JDBC和ODBC的API。HS2是一个单独运行的一个服务租，包括基于thirft的hive服务(TCP或者HTTP)以及一个jetty的web UI

# HS2 Architecture
基于thirft的hive服务是HS2的核心，它的职责主要是服务客户端过来的hive查询。 thirft服务是一个RPC的框架，它主要有四层，server, transport, protocol 和processor。

## Server
HS2使用一个```TThreadPoolServer```来实现TCP模型，使用```jetty server ```来实现HTTP模式

```TThreadPoolServer```会为每一个TCP连接申请一个worker线程。 线程与tcp连接一直关联，及时是在空闲的情况下。所以当并发连接的情况下, 会有大量的线程，这样会造成潜在的性能问题。 在未来，HS2 会采用另一种方式来实现TCP模式，比如```TThreadedSelectorServer```

## Transport
当客户端和服务器端连接需要代理的时候(或者其他原因，如负载均衡，以及安全原因)，HTTP 模式就需要被使用了。 可以通过配置 ```hive.server2.transport.mode```来指定thrift service的传输模式， 默认为binary

## Protocol
协议的主要责任是执行序列化和饭序列化操作。 HS2当前使用 ```TBinaryProtocol```作为其thrift协议来序列化数据。 未来协议是可以被指定的，类如为了更好的性能，采用```TCompactProtocol```等

## Processor
Process用于处理请求的逻辑。 类如 ThriftCLIService.ExecuteStatement() 用于实现一个hive的查询

# HS2 依赖

## Metastore
metastore可以是内嵌的数据库，也可以是远程的数据库。 HS2 使用metastore中的metadata来解释查询

## Hadoop集群
用于准备物理的执行计划(基于多种执行引擎，如MapReduce， Tez， Spark)以及提交任务到hadoop集群上去执行

# JDBC Client
推荐客户端使用jdbc driver来连接HS2. 很多场景下(如hadoop Hue)，直接调用thrift client 和 间接的调用jdbc client

当查询的时候，一系列的api会被调用：
+ jdbc 客户端 调用 OpenSession API 获取一个 SessionHandle后，通过初始化一个传输连接，创建了一个hiveConnection。 session是在服务端被创建的
+ 当HiveStatement被执行的时候， thrift 客户端执行一个ExecuteStatement API， 在这个api中， SessionHandle的信息与查询信息一起被传输到服务端
+ HS2 服务器接收到请求，并从driver(一个CommandeProcessor)中获取查询的语法以及解释。driver在后台启动一个job，用于与hadoop通讯，并响应客户端。 这个是ExecuteStatement API的异步实现。 响应信息包括了服务端创建的OperationHandle。
+ 客户端使用OperationHandle来通知HS2来拉取查询结果

# 源码

## 服务端

+ **Thrift IDL file for TCLIService**: https://github.com/apache/hive/blob/master/service-rpc/if/TCLIService.thrift.
+ **TCLIService.Iface implemented by**: org.apache.hive.service.cli.thrift.ThriftCLIService class.
+ **ThriftCLIService subclassed by**: org.apache.hive.service.cli.thrift.ThriftBinaryCLIService and org.apache.hive.service.cli.thrift.ThriftHttpCLIService for TCP mode and HTTP mode respectively.
+ **org.apache.hive.service.cli.thrift.EmbeddedThriftBinaryCLIService class**: Embedded mode for HS2. Don't get confused with embedded metastore, which is a different service (although the embedded mode concept is similar).
+ **org.apache.hive.service.cli.session.HiveSessionImpl class**: Instances of this class are created on the server side and managed by an org.apache.accumulo.tserver.TabletServer.SessionManager instance.
+ **org.apache.hive.service.cli.operation.Operation class**: Defines an operation (e.g., a query). Instances of this class are created on the server and managed by an org.apache.hive.service.cli.operation.OperationManager instance.
+ **org.apache.hive.service.auth.HiveAuthFactory class**: A helper used by both HTTP and TCP mode for authentication. Refer to Setting Up HiveServer2 for various authentication options, in particular Authentication/Security Configuration and Cookie Based Authentication.

## 客户端

+ **org.apache.hive.jdbc.HiveConnection class**: Implements the java.sql.Connection interface (part of JDBC). An instance of this class holds a reference to a SessionHandle instance which is retrieved when making Thrift API calls to the server.
+ **org.apache.hive.jdbc.HiveStatement class**: Implements the java.sql.Statement interface (part of JDBC). The client (e.g., Beeline) calls the HiveStatement.execute() method for the query. Inside the execute() method, the Thrift client is used to make API calls.
+ **org.apache.hive.jdbc.HiveDriver class**: Implements the java.sql.Driver interface (part of JDBC). The core method is connect() which is used by the JDBC client to initiate a SQL connection.

## 客户端与服务端交互

+ **org.apache.hive.service.cli.SessionHandle class**: Session identifier. Instances of this class are returned from the server and used by the client as input for Thrift API calls.
+ **org.apache.hive.service.cli.OperationHandle class**: Operation identifier. Instances of this class are returned from the server and used by the client to poll the execution status of an operation.
