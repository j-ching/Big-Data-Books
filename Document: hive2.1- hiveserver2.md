title: hive2.1- hiveserver2
notebook: 技术相关
tags: hive, hive2.1, hiveserver2

[TOC]

# HiveServer2
HS2 是一个服务接口，供客户端提交查询并且返回结果。它比HiveServer1更有优势， 虽然hiveserver1与hiveserver2都是通过thrift server提供的服务，但是hiveserver1不支持多个客户端同时请求。 而HS2提供了多客户端使用和授权，而且提供额更加友好的类似于JDBC和ODBC的API。HS2是一个单独运行的一个服务租，包括基于thirft的hive服务(TCP或者HTTP)以及一个jetty的web UI

# HS2 Architecture
基于thirft的hive服务是HS2的核心，它的职责主要是服务客户端过来的hive查询。 thirft服务是一个RPC的框架，它主要有四层，server, transport, protocol 和processor。 
