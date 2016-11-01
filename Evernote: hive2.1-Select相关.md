title: hive2.1-Select相关
notebook: 技术相关
tags: hive

[TOC]

# Select Syntax

	[WITH CommonTableExpression (, CommonTableExpression)*]    (Note: Only available starting with Hive 0.13.0)
	SELECT [ALL | DISTINCT] select_expr, select_expr, ...
	  FROM table_reference
	  [WHERE where_condition]
	  [GROUP BY col_list]
	  [ORDER BY col_list]
	  [CLUSTER BY col_list
	    | [DISTRIBUTE BY col_list] [SORT BY col_list]
	  ]
	 [LIMIT number]

+ select 可以是union query的一部分，也可以是另一个query的subquery
+ table_reference 标识了query的输入。 可以是一个严格意义上的table， 一个view，一个join查询或者一个子查询
+ table 名和column 名大小写相同
+ 简单查询
	```SELECT * FROM t1```
  从hive0.13.0开始，from可选(如： select 1+1)

+ 获取当前的数据库

  ```
	SELECT 	current_database()
  ```

+ 当指定数据库之后，可以通过指定在table名前家database名 或者使用 USE 来查询不同的database

  ```
	use database_name;
	select query_specifications;
	use default;
   ```

## WHERE Clause

where是boolean表达式，支持数值操作和UDF

    SELECT * FROM sales WHERE amount > 10 AND region = "US"

## ALL and DISTINCT Clauses

All 和 DISTINCT 选项用于指明重复的数据是否需要返回。 默认为ALL，

    hive> SELECT col1, col2 FROM t1
    1 3
    1 3
    1 4
    2 5
    hive> SELECT DISTINCT col1, col2 FROM t1
        1 3
        1 4
        2 5
    hive> SELECT DISTINCT col1 FROM t1
        1
        2

## Partition Based Queries

table创建分区之后，查询可以在每个分区上运行， 而且只会扫描查询中涉及到的分区。

    SELECT page_views.*
    FROM page_views
    WHERE page_views.date >= '2008-03-01' AND page_views.date <= '2008-03-31'

也可以在join的on表达式中使用分区

    SELECT page_views.*
    FROM page_views JOIN dim_users
    ON (page_views.user_id = dim_users.id AND page_views.date >= '2008-03-01' AND page_views.date <= '2008-03-31')

## HAVING Clause

从hive0.7.0中开始支持hiving

    SELECT col1 FROM t1 GROUP BY col1 HAVING SUM(col2) > 10

与之等同的子查询如下

    SELECT col1 FROM (SELECT col1, SUM(col2) AS col2sum FROM t1 GROUP BY col1) t2 WHERE t2.col2sum > 10

## LIMIT Clause
决定查询返回的条数， 返回的条数是随机的。

    SELECT * FROM t1 LIMIT 5

+ top K 查询

    ```
    SET mapred.reduce.tasks = 1
    SELECT * FROM sales SORT BY amount DESC LIMIT 5
    ```

## REGEX Column Specification
在hive0.13.0之前的版本，默认支持column上的正在表达式， 在0.13.0之后，需要配置资源文件 ```hive.support.quoted.identifiers ``` 为 none

    SELECT `(ds|hr)?+.+` FROM sales
