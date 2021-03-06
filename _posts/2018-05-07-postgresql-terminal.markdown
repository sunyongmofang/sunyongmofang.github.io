---
layout: post
title:  "postgresql数据库命令行使用教程"
date:   2018-05-07 17:12:17 +0800
categories: postgresql
---

登录命令
```
psql -U 用户名 -d 数据仓库名
```

登陆数据库后通常会以如下格式展现
`tablebase=#`
或
`tablebase=>`
tablebase为当前数据库名，=#或=>代表是否为root权限
在此模式下可以直接编写sql语句和建表语句等

命令行常用操作如下：
```
\h 帮助命令，查看所有sql语句
\d 查看当前数据库中所有的表和视图
\d+ tablename 查看表或试图的详细信息，tablename为表名或视图名
\l 查看所有数据仓库
\c tablebase 切换数据仓库，tablebase为数据仓库名
\df 查看当前数据库中的存储过程
\sf function(argv...) 查看存储过程详细信息，参数为存储过程名，方法重载时应指明参数列表
\copy tablename from/to 'youpath' with csv 导出/入数据库表到csv
```

添加file_fdw模块，并创建服务
```
create extension file_fdw;
create server pg_file_server foreign data wrapper file_fdw;
```

创建database、用户和密码
```
create user firstlevel with password 'firstlevel';
create database level_data owner firstlevel;
```

创建表
```
create table table_name(column_name data_type,...);
```
table_name和column_name为实际表名和列名，data_type为实际数据类型，最后一列的数据类型后无“,”

创建索引
```
create index index_name on table_name using index_type (column_name,...);
```
index_name和table_name为实际索引名和表名，index_type为实际索引类型，column_name为要建立索引的列名，最后一个列名后无“,”

在某个表中添加新列

```
ALTER table table_name add COLUMN column_name data_type default default_value;
```
default与后面的值可以不添加