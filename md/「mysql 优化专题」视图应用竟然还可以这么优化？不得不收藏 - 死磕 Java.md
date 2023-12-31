> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1600271767)

> 当一个大型系统在建立时，会发现，数据库虽然可以存储海量的数据，可是一旦数据关系复杂，比如学生表（学号、姓名、年龄），学生成绩表（学号、科目、成绩），如需要姓名、科目、成绩组成关

转

 2023-03-28  阅读 (173)  点赞 (0)

原文地址：https://cloud.tencent.com/developer/article/1185462

> 当一个大型系统在建立时，会发现，数据库虽然可以存储海量的数据，可是一旦数据关系复杂，比如学生表（学号、姓名、年龄），学生成绩表（学号、科目、成绩），如需要姓名、科目、成绩组成关系，这样的情况我们选择创建一个新表是非常浪费资源的动作，为此，视图诞生了！

（1）什么是视图？

视图是基于 SQL 语句的结果集的可视化的表。

视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。视图并不在数据库中以存储的数据值集形式存在，而是存在于实际引用的数据库表中，视图的构成可以是单表查询，多表联合查询，分组查询以及计算 (表达式) 查询等。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。

（2）视图的优点：

a、简化查询语句（视图机制使用户可以将注意力集中在所关心地数据上。如果这些数据不是直接来自基本表，则可以通过定义视图，使数据库看起来结构简单、清晰，并且可以简化用户的的数据查询操作。）

b、可以进行权限控制

把表的权限封闭，但是开放相应的视图权限，视图里只开放部分数据列等。

c、大数据表分表的时候，比如某张表的数据有 100 万条，那么可以将这张表分成四个视图。

按照对 id 取余计算

d、用户能以多种角度看待同一数据：

使不同的用户以不同的方式看待同一数据，当许多不同种类的用户共享同一个数据库时，这种灵活性是非常必要的。

e、对重构数据库提供了一定程度的逻辑独立性：

视图可以使应用程序和数据库表在一定程度上独立。

（3）视图的缺点：

1）性能差：

把视图查询转化成对基本表的查询，如果这个视图是由一个复杂的多表查询所定义，那么，即使是视图的一个简单查询，sql server 也要把它变成一个复杂的结合体，需要花费一定的时间。

2）修改限制：

当用户试图修改试图的某些信息时，数据库必须把它转化为对基本表的某些信息的修改，对于简单的试图来说，这是很方便的，但是，对于比较复杂的试图，可能是不可修改的。

（4）视图使用场景（其实就是需要用到视图上面的几个优点的时候）：

1） 需要权限控制的时候。

2）如果某个查询结果出现的非常频繁，就是要经常拿这个查询结果来做子查询，使用视图会更加方便。

3）关键信息来源于多个复杂关联表，可以创建视图提取我们需要的信息，简化操作；

（5）视图的分类：

1）关系视图：

它属于数据库对象的一种，也就是最常见的一种关联查询；

2）内嵌视图：

它不属于任何用户，也不是对象，创建方式与普通视图完全不同，不具有可复用性，不能通过数据字典获取数据；

3）对象视图：

它是基于表对象类型的视图，特性是继承、封装等可根据需要构建对象类型封装复杂查询 (官方: 为了迎合对象类型而重建数据表是不实现的)；

4）物化视图：

它主要用于数据库的容灾 (备份)，实体化的视图可存储和查询，通过 DBLink 连接在主数据库物化视图中复制，当主库异常备库接管实现容灾；

1、创建视图

1.  create or replace view v_test asselect * fromuser;

加上 OR REPLACE 表示该语句还能替换已有的视图

2、调取视图

1.  select * from v_test;

3、修改视图

1.  alter view v_test asselect * from user1;

4、删除视图

1.  drop view if exists v_test;

5、查看视图

1.  show tables;

视图放在 information_schema 数据库下的 views 表里

6、查看视图的定义

show table status from companys like'v_test';

a、Merge：合并的执行方式，每当执行的时候，先将我们的视图的 sql 语句与外部查询视图的 sql 语句，混合在一起，最终执行。

b、Temptable：临时表模式，每当查询的时候，将视图所使用的 select 语句生成一个结果的临时表，再在当当前临时表内进行查询。

（1）修改操作时要非常非常小心，不然不经意间你已经修改了基本表里的多条数据；

（2）视图中的查询语句性能要调到最优；

（3）虽说上面讲到，视图有些是可以修改的。但是更多的是禁止修改视图。

对于可更新的视图，在视图中的行和基表中的行之间必须具有一对一的关系或者特殊的没有约束的一对多字段。还有一些特定的其他结构，这类结构会使得视图不可更新。

不可更改的情况如下：视图中含有以下的都不可被修改了。

（一）聚合函数（SUM(), MIN(), MAX(), COUNT() 等）。

（二）DISTINCT。如下错误。

![](http://image.skjava.com/article/series/mysql/202303282258464631.png)

（三）GROUP BY

（四）HAVING

（五）UNION 或 UNION ALL

（六）位于选择列表中的子查询

（八）FROM 子句中的不可更新视图

（九）WHERE 子句中的子查询，引用 FROM 子句中的表。

（十）ALGORITHM = TEMPTABLE（使用临时表总会使视图成为不可更新的）。

> 今天，视图的应用就讲到这里，觉得有收获的同学可以收藏关注。本号内有多个专题，如【数据结构】、【netty 专题】、【dubbo 专题】、【mysql 优化专题】、【redis 专题】、【高并发专题】等优质好文。一起学习，共同进步。