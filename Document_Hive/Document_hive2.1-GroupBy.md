[TOC]

# Group By Syntax

	groupByClause: GROUP BY groupByExpression (, groupByExpression)*
	groupByExpression: expression
	groupByQuery: SELECT expression (, expression)* FROM src groupByClause?

groupByExpression的列需要通过名称来指定，不能使用位置编号。但是从hive0.11.0开始，可以设置参数 ```hive.groupby.orderby.position.alias=true``` 来使用位置标号(默认为false)

### Simple Examples

计算表的行数

	SELECT COUNT(*) FROM table2;

在不包含HIVE-287的版本中，需要使用COUNT(1)来代替COUNT(*)

按性别来统计用户的数量

	INSERT OVERWRITE TABLE pv_gender_sum
	SELECT pv_users.gender, count (DISTINCT pv_users.userid)
	FROM pv_users
	GROUP BY pv_users.gender;

多种聚合操作可以同时操作，但是，不能有俩种聚合操作包含不同的DISTINCT列。 如下的操作是可以的，因为count(DISTINCT) 和 sum(DISTINCT) 指向了相同的列

	INSERT OVERWRITE TABLE pv_gender_agg
	SELECT pv_users.gender, count(DISTINCT pv_users.userid), count(*), sum(DISTINCT pv_users.userid)
	FROM pv_users
	GROUP BY pv_users.gender;

然而，需要的查询是不允许的， 我们不允许在同一查询中包含不同的DISTINCT表达式

	INSERT OVERWRITE TABLE pv_gender_agg
	SELECT pv_users.gender, count(DISTINCT pv_users.userid), count(DISTINCT pv_users.ip)
	FROM pv_users
	GROUP BY pv_users.gender;

### Select statement and group by clause

使用group by时， select 只能包含group by 中的字段， 当然，select 中可以包含任意的聚合函数

例如：

	CREATE TABLE t1(a INTEGER, b INTGER);

group by 查询

	SELECT
	   a,
	   sum(b)
	FROM
	   t1
	GROUP BY
	   a;

以上sql正常，因为select包含了groupby中的a 和一个聚合函数sum

	SELECT
	   a,
	   b
	FROM
	   t1
	GROUP BY
	   a;

以上sql不能正常运行，因为b不在groupby中

# Advanced Features

### Multi-Group-By Inserts
aggregations 和 select 的输出可以进一步被输入到表中或者hdfs文件中(需要使用hdfs组件)

	FROM pv_users
	INSERT OVERWRITE TABLE pv_gender_sum
	  SELECT pv_users.gender, count(DISTINCT pv_users.userid)
	  GROUP BY pv_users.gender
	INSERT OVERWRITE DIRECTORY '/user/facebook/tmp/pv_age_sum'
	  SELECT pv_users.age, count(DISTINCT pv_users.userid)
	  GROUP BY pv_users.age;


### Map-side Aggregation for Group By

```hive.map.aggr ``` 来控制如果做聚合， 默认为false。 当设置为true时，hive将在map阶段执行第一阶段的聚合操作。 这样通常会提高效率，但是需要更大的内存。

	set hive.map.aggr=true;
	SELECT COUNT(*) FROM table2;

### Grouping Sets, Cubes, Rollups, and the GROUPING__ID Function

	从hive0.10.0开始，添加了Grouping sets, Cube, rollup操作，以及 Group_ID 方法

##### Grouping sets clause
Grouping 集合分区让我们可以在相同的数据集下，制定多个group by参数。grouping set分区在逻辑上可以表示为多个group by子句的union。
表格1 展现了几种相同的描述方式 ***在grouping sets中，一个空的set()会被解释为整个聚合操作***

| grouping sets | group by  |
|---------------|-----------|
| SELECT a, b, SUM(c) FROM tab1 GROUP BY a, b GROUPING SETS ( (a,b) ) |  SELECT a, b, SUM(c) FROM tab1 GROUP BY a, b |
|SELECT a, b, SUM( c ) FROM tab1 GROUP BY a, b GROUPING SETS ( (a,b), a)| SELECT a, b, SUM( c ) FROM tab1 GROUP BY a, b UNION SELECT a, null, SUM( c ) FROM tab1 GROUP BY a|
|SELECT a,b, SUM( c ) FROM tab1 GROUP BY a, b GROUPING SETS (a,b)|  SELECT a, null, SUM( c ) FROM tab1 GROUP BY a union SELECT null, b, SUM( c ) FROM tab1 GROUP BY b|
|SELECT a, b, SUM( c ) FROM tab1 GROUP BY a, b GROUPING SETS ( (a, b), a, b, ( ) )| SELECT a, b, SUM( c ) FROM tab1 GROUP BY a, b union SELECT a, null, SUM( c ) FROM tab1 GROUP BY a, null union SELECT null, b, SUM( c ) FROM tab1 GROUP BY null, b union SELECT null, null, SUM( c ) FROM tab1|







##### Cubes and Rollups
WITH CUBE/ROLLUP 只能用于Group by环境下， Cube 用group by的列集创建了一个具有所有可能组合的子集合。一旦计算了维度集合的cube， 我们就能够获取这些维度的所有可能的聚合操作

|         cube         |       group by       |
|----------------------|----------------------|
| GROUP BY a, b, c WITH CUBE | GROUP BY a, b, c GROUPING SETS ((a,b,c), (a, b), (b, c), (a, c), (a), (b), (c), ())


|         rollup         |       group by       |
|----------------------|----------------------|
| GROUP BY a, b, c with ROLLUP | GROUP BY a, b, c GROUPING SETS ((a,b,c), (a, b), (a), ())

###### ***hive.new.job.grouping.set.cardinality***
grouping sets/rollup/cubes 都是导致一个mr的任务被加载，类如： ```select a, b, c, count(1) from T group by a, b, c with rollup;``` ， 每行数据都是生成四行 (a,b,c), (a,b,null), (a, null, null), (null,null,null), 当table T 容量很大的时候，在mr的过程中就会造成数据膨胀， map端的聚合将不能完成。

这个参数决定了hive是否需要增加一个mr的job。如果group set的基础远大于这个值，则hive就会添加一个附加的mr在原始的数据上，用于削减数据量。


