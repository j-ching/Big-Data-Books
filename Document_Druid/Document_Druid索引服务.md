[TOC]

对于索引服务的配置，可以查看[索引服务]

  


索引服务是一个高性能呢，分布式服务，它用来运行相关联的索引任务。 索引服务可以创建或者删除druid的segments。索引服务有master/slave的架构。

索引服务由如下几个部分组成：

  1. 一个可以运行单独任务的peon component
  2. 一个middle manager 用于管理peon
  3. 一个overload 用于将任务分发到middle minager

overloads 和 middle managers 可以用于在相同的节点上或者多个节点上， 这时middle managers和penons 运行在相同的节点上

  


Indexing Service Overview

  Overlord Node

overload node的职责是用来接收任务，分发任务，创建任务锁，以及给调用者返回状态信息。 overload 可以配置运行在俩种模式下- local 和 remot。 

  1. 在local模式下， overload 也负责为执行的任务创建peons。当overload运行在local模式下时， 所有的middle manager 和 peon 配置信息都必须提供。 local模式用于简单的工作流。
  2. 在remote模式下，overload和middle manager 运行在俩个不同进程中，也可以运行在俩台不同的服务器。 如果你想将索引服务作为一个独立的endpoint 开放给所用的druid 来创建索引， 推介使用remote模式 。

  


Submitting Tasks and Querying Task Status

  1. 提交任务：任务以json 对象的形式被提交到overload node。 可以提交post请求给  
`http://<OVERLORD_IP>:<port>/druid/indexer/v1/task`将返回提交任务的taskid

  2. 关闭任务： 通过提交post请求给`http://<OVERLORD_IP>:<port>/druid/indexer/v1/task/{taskId}/shutdown`
3. 任务状态： 提交get请求`http://<OVERLORD_IP>:<port>/druid/indexer/v1/task/{taskId}/status`
4. 查看任务的segments： 提交get请求`http://<OVERLORD_IP>:<port>/druid/indexer/v1/task/{taskId}/segments`

Overlord Console

overload控制台可以看到延迟的任务，运行的任务，可用的worder以及当前worker的创建及终止，地址：

`http://<OVERLORD_IP>:<port>/console.html`


