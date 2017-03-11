[TOC]

# Order, Sort, Cluster, and Distribute By
用于描述select字句的关键词有``` order by ```, ```sort by ```,```cluster by ``` 和 ```distribute by ```

## order by
与sql中的```order by```相似,

+ colOrder: ( ASC | DESC )
+ colNullOrder: (NULLS FIRST | NULLS LAST)           -- (Note: Available in Hive 2.1.0 and later)
+ orderBy: ORDER BY colName colOrder? colNullOrder? (',' colName colOrder? colNullOrder?)*
+ query: SELECT expression (',' expression)* FROM src orderBy

```order by ``` 在strict模式(```hive.mapred.mode=strict```)下有一些限制，就是order by 字句后边需要跟一个 ```limit ```字句， 当在非strict模型下，limit字句并不是必须的。 原因是为了能个达到全部的结果做排序，只能有一个reduce来排序所有的输出，当输出的数据量太大的时候，reduce会执行比较慢。

注： order by 的列需要通过列明来指定，不能是位置标号。 在hive0.11.0之后，  ```hive.groupby.orderby.position.alias``` 设置为true，就可以使用位置标号了。

默认的排序是asc(升序)

在hive2.1.0之后，可以在order by字句中指明null值的排序，Null的排序是asc为```NULLS FIRST```,
Null排序为desc则为```NULLS LAST```

## sort by
与```order by```相同

+ colOrder: ( ASC | DESC )
+ sortBy: SORT BY colName colOrder? (',' colName colOrder?)*
+ query: SELECT expression (',' expression)* FROM src sortBy

```sort by``` 是在将数据送入reduce之前，进行排序。 排序的顺序取决于列的类型，如果列为数字类型，则按数字排序，如果列为字符类型， 则按字符类型排序

### sort by 与 order by的不同
sort by支持对每个reduce的数据进行排序， 他们之间的不同在于， ```order by``` 保证了在整个输出的数据是有序的，而```sort by ``` 只是对每个reduce的数据进行排序，如果有多个reduce， ```sort by ```只能保证最后结果的部分有序


