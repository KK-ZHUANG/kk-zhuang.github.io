---
layout: post
title:  "MySQL JOIN类型及其使用"
date:   2018-09-24 00:00:00 +0800
categories: MySQL
---
## MySQL JOIN类型及其使用

### 内连接
内连接inner join可以不使用on，此时和交叉连接等价。  
使用方法：
```sql
select * from table_a a join table_b b on a.id=b.id;
select * from table_a a inner join table_b b on a.id=b.id;
上面两种方法，与下面等价
select * from table_a a,table_b where a.id=b.id;
通常省略inner，使用第一种。
```

### 外连接
外连接必须要有on。

* #### 左外连接  
获取左表所有行，右表不匹配的，用null填充。  
使用方法：
```sql
select * from table_a a left join table_b b on a.id=b.id;
select * from table_a a left outer join table_b b on a.id=b.id;
通常省略outer，使用第一种。
```

* #### 右外连接  
获取右表所有行，左表不匹配的，用null填充。  
使用方法：
```sql
select * from table_a a right join table_b b on a.id=b.id;
select * from table_a a right outer join table_b b on a.id=b.id;
通常省略outer，使用第一种。
```

* #### 全外连接
即左外连接与右外连接的并集，经测试，MySQL不支持。  
使用方法：
```sql
select * from table_a a full join table_b b on a.id=b.id;
select * from table_a a full outer join table_b b on a.id=b.id;
通常省略outer，使用第一种。
```
MySQL中可以使用union实现。  
使用方法：
```sql
select * from table_a a left join table_b b on a.id=b.id
union
select * from table_a a right join table_b b on a.id=b.id;
```

### 交叉连接
cross join 即笛卡尔积。

使用方法
```sql
select * from table_a a cross join table_b;
此时等价于
select * from table_a a inner join table_b;
```

有资料显示cross join与inner join的区别在于不能加on，但实际在MySQL中测试，可以使用on，即
```sql
select * from table_a a cross join table_b on a.id=b.id;
此时等价于
select * from table_a a inner join table_b b on a.id=b.id;
```
很少使用，多用inner join代替。

### 其他

#### 关于连接中笛卡尔积的计算顺序
MySQL中，JOIN的笛卡尔积计算顺序，似乎是以右表或行数多的表为主导的。
例如：  

__table_a__

id | name
:-: | :-:
1 | n1
2 | n2

__table_b__

id | age
:-: | :-:
1	| 18
2	| 19

此时
```sql
select * from table_a join table_b;
```
结果为

id | name | id | age
:-: | :-: | :-: | :-:
1 | n1 | 1 | 18
2 | n2 | 1 | 18
1 | n1 | 2 | 19
2 | n2 | 2 | 19

```sql
select * from table_a join table_b;
```
结果为

id | age | id | name
:-: | :-: | :-: | :-:
1 | 18 | 1 | n1
2 | 19 | 1 | n1
1 | 18 | 2 | n2
2 | 19 | 2 | n2

__即当两表行数相等时，以右表为主导。__

而当  
__table_b__

id | age
:-: | :-:
1	| 18
2	| 19
3 | 20

此时
```sql
select * from table_a join table_b;
```
结果为

id | name | id | age
:-: | :-: | :-: | :-:
1 | n1 | 1 | 18
2 | n2 | 1 | 18
1 | n1 | 2 | 19
2 | n2 | 2 | 19
1 | n1 | 3 | 20
2 | n2 | 3 | 20

而
```sql
select * from table_b join table_a;
```
时，结果为

id | age | id | name
:-: | :-: | :-: | :-:
1 | 18 | 1 | n1
1 | 18 | 2 | n2
2 | 19 | 1 | n1
2 | 19 | 2 | n2
3 | 20 | 1 | n1
3 | 20 | 2 | n2

__即两表行数不等时，以行数多的表为主导。__
