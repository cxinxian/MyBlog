---
layout: post
title: SQL备忘录一
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
1、尽量不使用游标，因为效率差；
2、如果操作数据超过1w,应当改写游标


### 三、存储过程
> **概念：**
存储过程Procedure是一组为了完成特定功能的SQL语句集合，经编译后存储在数据库中，用户通过指定存储过程的名称并给出参数来执行。
存储过程中可以包含逻辑控制语句和数据操纵语句，它可以接受参数、输出参数、返回单个或多个结果集以及返回值。

**作用及优点：**
* 增强数据的完整性
* 复杂业务规则和约束的一致实现
* 模块化设计
* 可维护性
* 降低网络通讯量
* 提高执行效率
* 较强的安全性

由于存储过程在创建时即在数据库服务器上进行了编译并存储在数据库中，所以存储过程运行要比单个的SQL语句块要快。同时由于在调用时只需用提供存储过程名和必要的参数信息，所以在一定程度上也可以减少网络流量、简单网络负担。

**具体使用**

例子：更新用户密码
```
create proc update_user_pwd
	@usr varchar(50),				 --输入变量，接收用户名
	@old_pwd varchar(500),			--输入变量，接收旧密码
	@new_pwd varchar(500),			--输入变量，接收新密码
	@res int Output				   --输出变量，输出结果标志 0：成功； -1：用户名错误； -2：旧密码错误
As
if exists(select * from tb_user where username = @usr)
	if exists(select * from tb_user where username = @usr and passworld = @pwd)
		begin
			update tb_user set passworld = @new_pwd where username = @usr
			set @res = 0
		end
	else
		select @res = -2
else
	select @res = -1
return @res
```

实现分页查询：
```
--使用row_number函数分页
if OBJECT_ID (N'PROC_SELECT_BY_PAGE', N'P') IS NOT NULL
    drop procedure PROC_SELECT_BY_PAGE;
gO
create procedure PROC_SELECT_BY_PAGE
    @startIndex int,
    @endIndex int
as
    select  * from (select id,name,age,sex,row_number() over(order by id desc) as rownumber from tb_user) as Temp
    where Temp.rownumber between @startIndex AND @endIndex
go



--使用top 分页查询
使用row_number函数分页
if OBJECT_ID (N'PROC_SELECT_BY_PAGE_top', N'P') IS NOT NULL
    drop procedure PROC_SELECT_BY_PAGE_top;
gO
create procedure PROC_SELECT_BY_PAGE_top
    @pageIndex int,
    @pageSize int
as
    select top(pageSize) * from tb_user where id >= (select max(id) from (select top(pageSize*(pageIndex-1)+1 from tb_user) id from tb_user order by id)as Temp)
go

```
