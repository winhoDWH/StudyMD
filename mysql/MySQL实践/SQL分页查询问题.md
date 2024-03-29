### 问题描述
当表A数据量大的时候（假设有30多万条数据），需要分页查询某一张表A的所有字段的时候，使用两种查询方式，查询速度差异大；
1. 第一种方式：`SELECT * FROM A WHERE passtime between '2020-01-01 00:00:00' and '2021-01-01 23:59:59' LIMTI 300000, 5`(查询第30万条数据后5条数据)查询需要5秒；
2. 第二种方式：`SELECT * FROM A a1 right join (SELECT ID FROM A a2 WHERE passtime between '2020-01-01 00:00:00' and '2021-01-01 23:59:59' LIMTI 300000, 5) on a1.ID = a2.ID`;(使用**右连接**)查询不到1秒；

### 原理结论
1. 上面表A有两个索引，时间`passtime`和`ID`，其中`ID`是主键，作为聚簇索引；而`passtime`作为普通索引，索引指向的是一个对应记录的`ID`号，所以根据该索引查询时，需要用对应的ID号反查整条记录；加上sql执行顺序是先执行select部分再执行limit语句部分，所以可以总结出两种方式具体的执行方式；
    * 第一种方式：先根据`passtime`索引找出符合规则的记录**对应的ID号**，由于是`select *`，所以先将所有符合规则的记录对应记录的所有信息都反查出来，这里由于分页所以可能反查出30000条以上的数据，最后在对这个庞大的集合进行分页返回；
    * 第二种方式
2. 涉及的相关原理：sql执行顺序、聚簇索引、页、连接查询原理

### 相关原理解释
#### sql执行顺序
1. 