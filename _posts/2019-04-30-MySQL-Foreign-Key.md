---
layout: post
title:  "MySQL Foreign Key"
date:   2019-04-30 00:55:39 +0800
categories: MySQL
---
## MySQL 外键

代码均在MySQL 5.7.24 版本测试

### 创建

* #### 语法

``` sql
[CONSTRAINT [symbol]] FOREIGN KEY
    [index_name] (col_name, ...)
    REFERENCES tbl_name (col_name,...)
    [ON DELETE reference_option]
    [ON UPDATE reference_option]

reference_option:
    RESTRICT | CASCADE | SET NULL | NO ACTION | SET DEFAULT
```

* #### 建表时创建

``` sql
create table parent (
  id int auto_increment primary key,
  uuid char(32) not null default '',
  name varchar(255) not null default '',
  unique (uuid)
);

create table child (
  id int auto_increment primary key,
  uuid char(32) not null default '',
  parent_uuid char(32) not null default '',
  name varchar(255) not null default '',
  unique (uuid),
  key (parent_uuid),
  constraint FK_parent_uuid_child_parent_uuid foreign key (parent_uuid) references parent (uuid)
);
```

* #### 为已存在的表创建

``` sql
alter table child add constraint FK_parent_uuid_child_parent_uuid foreign key (parent_uuid) references parent (uuid);
```

### 删除

``` sql
alter table child drop foreign key FK_parent_uuid_child_parent_uuid;
```

### 级联更新、删除

* #### RESTRICT
  默认选项，不允许级联更新、删除
* #### CASCADE
  父表 更新|删除 时，会 更新|删除 子表中对应的行，即级联更新、删除
* #### SET NULL
  父表 更新|删除 时，子表对应的外键会被更新为NULL，要求子表外键列允许为NULL
* #### NO ACTION
  MySQL中，等同RESTRICT
* #### SET DEFAULT
  父表 更新|删除 时，子表对应的外键会被更新为DEFAULT的值，InnoDB不支持

### 一些问题
1. 外键相关的列必须要有索引，父表没有则必须创建，子表没有则会自动创建一个和外键名相同的索引（旧版本需要显式创建）。
``` sql
create table child (
  id int auto_increment primary key,
  uuid char(32) not null default '',
  parent_uuid char(32) not null default '',
  name varchar(255) not null default '',
  unique (uuid),
  constraint FK_parent_uuid_child_parent_uuid foreign key (parent_uuid) references parent (uuid)
);
show CREATE table child;
//结果
CREATE TABLE `child` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `uuid` char(32) COLLATE utf8mb4_bin NOT NULL DEFAULT '',
  `parent_uuid` char(32) COLLATE utf8mb4_bin NOT NULL DEFAULT '',
  `name` varchar(255) COLLATE utf8mb4_bin NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uuid` (`uuid`),
  KEY `FK_parent_uuid_child_parent_uuid` (`parent_uuid`),
  CONSTRAINT `FK_parent_uuid_child_parent_uuid` FOREIGN KEY (`parent_uuid`) REFERENCES `parent` (`uuid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```

2. 支持组合索引，但要求符合最左前缀原则。  
  如父表索引为
  ``` sql
  UNIQUE KEY `uuid` (`uuid`,`name`)
  ```
  则字表外键
  ``` sql
  constraint FK_parent_uuid_child_parent_uuid foreign key (parent_uuid,name) references parent (uuid,name)  //√
  constraint FK_parent_uuid_child_parent_uuid foreign key (name) references parent (name)                   //×
  ```

3. 如有多个索引符合创建外键的要求，可以被删除，但至少要保留一个。  
  如字表索引和外键为
  ``` sql
  KEY `parent_uuid` (`parent_uuid`),
  KEY `parent_uuid_2` (`parent_uuid`,`name`),
  CONSTRAINT `FK_parent_uuid_child_parent_uuid` FOREIGN KEY (`parent_uuid`) REFERENCES `parent` (`uuid`)
  ```
  删除一个索引，没有问题
  ``` sql
  alter table child drop key parent_uuid;   //√
  ```
  再继续删除索引，出错
  ``` sql
  alter table child drop key parent_uuid_2; //×
  //结果
  [Err] 1553 - Cannot drop index 'parent_uuid_2': needed in a foreign key constraint
  ```

4. 外键不支持索引前缀。  
  如父表索引为
  ``` sql
  UNIQUE KEY `uuid` (`uuid`(16))
  ```
  则子表无法建立索引
  ``` sql
  constraint FK_parent_uuid_child_parent_uuid foreign key (parent_uuid) references parent (uuid)  //×
  //结果
  [Err] 1215 - Cannot add foreign key constraint
  ```

### 参考资料
>[https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html){:target="_blank"}
