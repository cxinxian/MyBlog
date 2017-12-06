---
layout: post
title: SQL备忘录
date: 2017-11-30 23:12:27
categories:
tags:
	SQL
toc: true
description: 点点滴滴就是积累
---

公司开始每周一次分享会，老前辈分享了些SQL方面的知识，值得记录。
于是我又重新拾起我的hexo。
回看这时间跨度也太长了...[手动捂脸]
感觉我错过来太多精彩没有记录下来，罪过罪过，还好现在也不算太晚。。。

<!-- more -->

### 一、SQL优化

1、对查询优化，尽量避免全表扫描，优先考虑在`where`及`order by` 涉及的列上建立索引

2、避免在`where`字句中对字段进行`null、!=、<>` 判断或者or连接条件，否则引擎放弃使用索引而进行全表扫描。
    eg:
    `select * from A where column is null or column=0`
    改成:
    `select * from A where  isnull(column,0)`

    尽量不给字段留NULL，尽可能使用NOT NULL；
    因为当字段为 char(100) 型，建立字段时，空间就固定，不管是否有值(包含NULL)，都占用100个字符空间；如果是varchar可变长字段，null不占用空间。

3、对in 和 not in 要慎用，否则也会导致全盘扫描
对于连续数据，能用between 就不用in
    eg:
    `select * from A where num in (1,2,3)`
    改成：
    `select * from A where num between 1 and 3`
大多时候用exists 代替 in
`select num from A where num in(select num from B)`
改为：
`select num from A where exists(select 1 from B where num=A.num)`

4、索引的创建要与应用结合考虑，建议大的OLTP表不要超过6个索引。

5、尽可能的使用索引字段作为查询条件，尤其是聚簇索引，必要时可以通过index index_name来强制指定索引   如:
```
select id from t where num=@num 
//可以改为强制查询使用索引：
select id from t with(index(索引名)) where num=@num
```
在使用索引字段作为条件时，如果该索引是联合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用。。
要注意索引的维护，周期性重建索引，重新编译存储过程。

### 二、SQL使用规范
1、尽量使用游标，因为效率差；
2、如果操作数据超过1w,应当改写游标




### 三、存储过程

