[TOC]

在druid集群中，coordinate nodes 通过规则来决定什么数据需要被load，或者什么数据需要从集群删除。 规则通过数据保存，并发送到coordinator console(http://coordinator_ip:port)来执行

规则指定了：

  1. 指定了segments如何被分配到不同的historical node层， 以及每一个historical node层需要保存多少个segments的副本
  2. 指定了什么时候segments需要从整个druid集群中删除

coordinate 从metadatastorage中加载一系列的规则。规则中可以指定一个datasource，配置一个默认的规则集合。 规则按顺序被读取并导入。 coordinate 将循环扫面所有可用的segments，并用第一个规则匹配所有的segments，每个segments只能匹配一条规则。

注意，规则的配置可以通过coordinate控制台，也可以在coordinate node上通过http endpoint来配置规则

当规则更新后，等下coordinate下一次运行时才会生效。

