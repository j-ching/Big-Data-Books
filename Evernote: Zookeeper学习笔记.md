---
title: Zookeeper 学习笔记
notebook: 技术相关
tags:zookeeper
---


+ Zookeeper Data Model
+ Zookeeper Sessions
+ Zookeeper Watches
+ Consistency Guarantees



znode mode :

+ persistent 当客户端断开连接后，znode不会被自动删除
+ persistent_sequential 当客户端断开连接后，znode不会被自动删除, 并且它的名字将被添加一个单调递增的数字
+ ephemeral 当客户端断开连接后，znode会被自动删除
+ ephemeral_sequential 当客户端断开连接后，znode会被自动删除,  并且它的名字将被添加一个单调递增的数字