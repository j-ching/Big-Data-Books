---
title: hive2.1-GroupBy
notebook: 技术相关
tags:hive
---

# Group By Syntax

	groupByClause: GROUP BY groupByExpression (, groupByExpression)*
	groupByExpression: expression
	groupByQuery: SELECT expression (, expression)* FROM src groupByClause?

groupByExpression的列需要通过名称来指定，不能使用位置编号。但是从hive0.11.0开始，可以设置参数hive.groupby.orderby.position.alias=true 来使用位置标号(默认为false)

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

# Enhanced Aggregation, Cube, Grouping and Rollup





详细如下
