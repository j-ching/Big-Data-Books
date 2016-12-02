# 引言

在闲暇之余，对自己的知识框架做了整理，总结经验，与大家分享

###【数据方向】

##### 数据收容
apache的chukwa，facebook的scribe， cloudera的flume以及ELK stack(归属于Elastic.co公司)组合中的logstash等常用数据收容框架的使用及架构设计； databus设计思路及目前比较流行的数据总线(如linked的kafka以及Ali的TT)使用和实现；利用数据总线实现大吞吐高并发的数据收容框架；
##### 数据统计分析
数据的清洗，处理；数据仓库的设计和构建；OLAP 服务的实现；较为流行的OLAP框架(如mondrian, Presto(facebook)，kylin(apache),  Lylin(ebay)、pinot(linkedin)和druid等)的了解和使用；深入理解OLAP框架的设计和实现；hadoop及hadoop生态系统的了解和使用，包括分布式文件系统，如GFS、HDFS等，分布式计算框架，如hadoop的mr离线结算框架，spark的批处理计算框架以及storm的流失计算框架等， 分布式资源管理框架，如yarn，mesos等，分布式调度框架，如zk等；hive，hbase，impala，pig，sqoop，elk等周边工具的研究和使用。分布式工作流框架，如azkaban, ozzie 的理解和使用。
##### 数据挖掘
接触到了一些常用的机器学习的算法，包括：逻辑回归，线性SVM，随机森林，梯度渐进决策树(回归)和地图渐进决策树（分类），朴素贝叶斯，K近邻，线性回归等。（由于本人愚笨，目前只停留在简单的使用)
其他： 涉及到一些gis相关的知识点，如空间索引

###【服务方向】

+ RPC服务的设计和使用
+ SOCKET服务在实时系统中的应用
+ Actor模型的深入理解，以及akka框架的应用
+ 分布式数据同步中间件的设计，以及datax的实现架构及使用
+ 分布式缓存，分布式一致性的设计思路，以及tair的实现架构及使用
+ 分布式配置中心的实现，以及zk在分布式服务中的使用场景
+ 分布式环境下的负载均衡
+ 分布式场景下的事务JTA

###【前端方向】

+ 前段开发中的AMD，CMD模式
+ HTML5+CSS3
+ nodejs 前端服务架构(npm + gulp/grunt + less + pm2)
+ socketio 实现实时展现
+ ES6新特征的引入， 如generator， promise等
+ bootstrap 开源框架的定制化改造，d3 图形引擎封装可视化图形工具
+ 实时计算 + socket服务 + socketIO 封装实时组件，应用场景包括服务接口监控，实时大屏等
