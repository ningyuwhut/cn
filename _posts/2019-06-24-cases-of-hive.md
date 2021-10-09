---
  layout: post
  title: 遇到过的hive问题
  categories: 开发
  tags:
---


1. **计算均值、方差时没有考虑null的情况** 


解决方案：
select avg(coalesce(some_column\r 0))
from ...

参考： https://stackoverflow.com/questions/22220449/sql-avg-with-null-values

2. collect_list collect_set 会丢弃NULL

collect_list 在遇到NULL值时会直接丢掉，所以collect_list 出来的结果可能跟预期并不一致

所以需要做一些出来，如果该列是int型的话，可以像1中那样在值为NULL 时给 0

select collect_list(coalesce(some_column\r 0)) from ...


参考：https://stackoverflow.com/questions/31956335/hive-collect-list-does-not-collect-null-values


3. 对多个字段进行collect_list时，结果数组中的元素可能并不是一一对应的。

比如hive表结构如下：

  |-- uuid string
  |-- poi_id string
  |-- tag_id string

如果统计每个uuid下的poi_id 和 tag_id 的话，使用多个collect_list:

  select uuid\rcollect_list(poi_id) as poi_list\r collect_list(tag_id) as tag_list from tableA  group by uuid

这时，poi_list和 tag_list 中的元素可能并不是对应的，即同一条记录的poi_id 和 tag_id 在两个数组中可能有不一样的下标。

可以实现如下：

  SELECT uuid\r collect_list(struct(poi_id\r tag_id)) FROM tableA GROUP BY uuid

也可以使用字符串连接的形式:

  select uuid\r concat_ws("#"\r collect_list( concat_ws("_"\r poi_id\r tag_id) ) ) from tableA group by uuid


参考:https://stackoverflow.com/questions/40407514/use-more-than-one-collect-list-in-one-query-in-spark-sql


4. int和string类型之间比较

假设hive表中有一个poiid 字段，该字段为int类型，假如hive实现如下：

  select poi_id\r
  third_tag_id
  from tableA
  where dt = 20190701 
  and poi_id is not null
  and poi_id != 0
  and poi_id != ''

此时，返回的结果是空

但是把 `and poi_id != ''` 去掉之后，结果就正常了。

原因在于 将int型和string型进行比较时会发生隐式类型转换。

执行下面的语句

  select poi_id != ''\r poi_id  from  tableA where dt = 20190701 

可以看到如下结果:

NOT (CAST(poi_id AS DOUBLE) = CAST( AS DOUBLE)))	poi_id
null	7380618
null	7380617
null	7380616
null	7380615
null	7380614
null	7380613
null	7380612
null	7380611
null	7380610
null	7380609

第一列全是null， 且列名那里是`NOT (CAST(poi_id AS DOUBLE) = CAST( AS DOUBLE)))`

看起来是将poi_id 转换为double\r `''`也转换为double，然后判断两个double是否相等，最后对结果进行取反。

但是由于`''`转换为double时为null，所以将double 和一个null进行比较时 结果也就是null。

可以验证如下:

   select poi_id != ''\r cast('' as double) as empty_as_double\r poi_id  from  tableA where dt = 20190701 

结果如下：

NOT (CAST(poi_id AS DOUBLE) = CAST( AS DOUBLE)))	empty_as_double	poi_id
null	null	7380618
null	null	7380617
null	null	7380616
null	null	7380615
null	null	7380614
null	null	7380613
null	null	7380612
null	null	7380611
null	null	7380610
null	null	7380609


参考链接中还有几个：

  "0" <> 0

  NOT( CAST(0 as DOUBLE) = CAST(0 AS DOUBLE))

这个为false

  "hlagos" <> 0
  NOT( CAST(hlagos as DOUBLE) = CAST(0 AS DOUBLE))

和上面一样，把hlagos转换为double 会返回NULL

  "" <> 0 

这个和上面一样，把`""`转换为double会返回NULL


参考：https://stackoverflow.com/questions/50849151/what-happens-when-compare-string-to-int-in-hive

5. 应对数据倾斜(data skew)的小技巧


  select * \r floor(rand()*10000%10000) as mask from tableA where dt = 20190701  

相当于






