# 实时计算框架



# flink 简介
![](media/15428867631690.jpg)

+ Stateful Stream Processing 有状态的数据处理， 通过ProcessFunciton 封装在上层的DS API里， 方便用户处理数据流。 大多数应用不会用到底层的API
+ DataStream/DataSetAPI  大多数都通过该API进行开发。这些api提供了常用的处理数据的方法， 比如用户自定义的transformation, join， aggregation, windows等。 DataStream API内嵌了底层的ProcessFunction， 方便用户做一些底层操作。 DataSet API 提供了一些额外的基于DataSet的原语，如loops/Iteration
+ Table API 是一个基于table的DSL， 可以基于表的概念来操作数据
+ SQL 是对Table API的上层抽象， 可以通过SQL语言来描述相关的任务逻辑


## 



# blink







# 计算平台






## 实时统计

## 异常检测



# 实时

