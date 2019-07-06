---
layout: post
title:  "MySQL Using Index"
date:   2019-07-06 00:00:00 +0800
categories: MySQL
---
## MySQL 索引的使用

### 种类

* 普通索引  
最基本的索引，没有任何限制。

* 唯一索引  
与普通索引的区别是，索引值必须唯一，但允许有空值。

* 主键索引  
与唯一索引的区别是，一个表只能有一个主键索引，且不允许有空值。

* 组合索引  
由多个字段组成的索引，可以是普通组合索引，唯一组合索引，主键组合索引（复合主键）。  
最左前缀原则。

### 创建索引

* 直接创建索引
``` sql
create index i_name on mytable(name); //普通索引
create unique index i_phone on mytable(phone(20)); //唯一索引
create index i_name_phone on mytable(name(20),phone(20)); //普通组合索引
create unique index i_name_phone on mytable(name(20),phone(20)); //唯一组合索引
```

* 创建表时创建索引
``` sql
create table mytable (
    id int auto_increment not null,
    name varchar(255),
    password varchar(255),
    phone varchar(255),
    email varchar(255),
    primary key(id), //主键索引
    index i_name (name), //普通索引
    unique [index] i_phone (phone(20)) //唯一索引
);
```

* 修改表结构创建
``` sql
alter table mytable add index i_name (name); //普通索引
alter table mytable add unique [index] i_phone (phone(20)); //唯一索引
```

### 删除索引
``` sql
drop index i_name on mytable;
或
alter table mytable drop index i_name;
```

### 其他操作

* 查看表的索引
``` sql
show index from mytable;
或
show keys from mytable;
```

* 查看索引使用情况
``` sql
explain + 查询语句;
```
