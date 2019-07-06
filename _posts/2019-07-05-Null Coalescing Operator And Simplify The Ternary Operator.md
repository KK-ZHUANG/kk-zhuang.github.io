---
layout: post
title:  "Null Coalescing Operator & Simplify The Ternary Operator"
date:   2019-07-05 00:00:00 +0800
categories: PHP
---
## NULL合并操作符和三元运算符简化

最近在学习Laravel，然后发现了一行代码
``` php
$_ENV['APP_BASE_PATH'] ?? dirname(__DIR__)
```
?? 之前并没有见过  
我心想，不应当，PHP里不应当有我不懂的东西，于是开始面向百度编程。  
输入 ??  
然后搜索出一堆问号表情包，此时我的内心：

![??](https://gss0.bdstatic.com/-4o3dSag_xI4khGkpoWK1HF6hhy/baike/w%3D268%3Bg%3D0/sign=1badc4a30c082838680ddb1280a2ce3c/8cb1cb1349540923e1860cc29958d109b2de499a.jpg)

好在皇天不负有心人，让我在PHP手册里找到了这玩意，叫做NULL合并操作符，以及一个三元运算符的简化写法。

### NULL 合并操作符

>[从左往右第一个存在且不为 NULL 的操作数。如果都没有定义且不为 NULL，则返回 NULL。PHP7开始提供。](https://www.php.net/manual/zh/language.operators.comparison.php)

手册说明感觉不太好理解，我的理解是：  
返回从左往右第一个不为 NULL 的操作数，如果全为NULL，则返回NULL。

示例代码
``` php
$a = $b ?? 'cc';
var_dump($a); //'cc'

$a = null ?? null ?? 'aa';
var_dump($a); //'aa'
```

这个操作符可以避免操作未定义变量时产生的警告。  
用来检查参数还挺好用。
``` php
$username = trim($_POST['username'] ?? exit('username is required!'));
```

### 三元运算符简化

示例代码
``` php
$a = 'a';
$b = $a ?: 'b';
var_dump($b); //'a'
```
