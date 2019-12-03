---
layout: post
title:  "Composer Autoload Source Code Analysis"
date:   2019-11-30 00:00:00 +0800
categories: PHP
---
## composer autoload源码分析 - 程序是如何找到它心爱的包包的

### 流程概览
以Laravel框架为例，项目目录为根目录，autoload流程如下
![流程图](https://processon.com/chart_image/5de22018e4b0df12b4ad3f5c.png)

### 源码分析
* 第一步  
略。

* 第二步  
略。

* 第三步  
我们直接从第三步开始，即autoload_real.php，这个real代表了这个文件才是真正的主菜，接下来我们逐个进行<s>哲♂学</s>分析。

调用的是getLoader方法，所以我们直接看此方法。
```php
if (null !== self::$loader) {
    return self::$loader;
}
```
单例模式。

```php
spl_autoload_register(array('ComposerAutoloaderInit55013298d242363653752f43b77cfdfc', 'loadClassLoader'), true, true);
self::$loader = $loader = new \Composer\Autoload\ClassLoader();
spl_autoload_unregister(array('ComposerAutoloaderInit55013298d242363653752f43b77cfdfc', 'loadClassLoader'));
```
实例化自动加载类。  
此处先将loadClassLoader方法注册为autoload方法，然后实例化ClassLoader作为autoload核心类，然后再注销了loadClassLoader方法。  
此处代码应该等同于下面代码  
```php
require __DIR__ . '/ClassLoader.php';
self::$loader = $loader = new \Composer\Autoload\ClassLoader();
```
因此源码写法我觉得有点多余，可能是为了更好的耦合性或者其他我没有考虑到的原因，因此采用了这种写法，但目前看起来效果应该是一样的。

```php
$useStaticLoader = PHP_VERSION_ID >= 50600 && !defined('HHVM_VERSION') && (!function_exists('zend_loader_file_encoded') || !zend_loader_file_encoded());
if ($useStaticLoader) {
    require_once __DIR__ . '/autoload_static.php';

    call_user_func(\Composer\Autoload\ComposerStaticInit55013298d242363653752f43b77cfdfc::getInitializer($loader));
} else {
    $map = require __DIR__ . '/autoload_namespaces.php';
    foreach ($map as $namespace => $path) {
        $loader->set($namespace, $path);
    }

    $map = require __DIR__ . '/autoload_psr4.php';
    foreach ($map as $namespace => $path) {
        $loader->setPsr4($namespace, $path);
    }

    $classMap = require __DIR__ . '/autoload_classmap.php';
    if ($classMap) {
        $loader->addClassMap($classMap);
    }
}
```
装载映射数组。  
根据不同的情况（PHP版本，是否HHVM等）分为静态初始化和接口初始化，作用是一样的，所以我们直接看第一个分支。
```php
require_once __DIR__ . '/autoload_static.php';
call_user_func(\Composer\Autoload\ComposerStaticInit55013298d242363653752f43b77cfdfc::getInitializer($loader));
```
引入autoload_static.php文件，并调用其getInitializer方法。其目的只有一个，将autoload的映射数组，绑定到核心类$loader中。  

接下来我们打开autoload_static.php，此时仿佛打开了一本新华字典，那么这里面的东西是怎么来的呢？  
在执行一些composer命令如require XXX，或者dump-autoload这一类的操作的时候，composer会按照一定的规则遍历各个文件，将file或类与其对应的文件路径解析成数组并保存到此文件，至于规则到底是什么样的规则，呃…，请参考PHP自动加载的四种方式，此处略。

接下来我们看getInitializer方法。
```php
public static function getInitializer(ClassLoader $loader)
{
    return \Closure::bind(function () use ($loader) {
        $loader->prefixLengthsPsr4 = ComposerStaticInit55013298d242363653752f43b77cfdfc::$prefixLengthsPsr4;
        $loader->prefixDirsPsr4 = ComposerStaticInit55013298d242363653752f43b77cfdfc::$prefixDirsPsr4;
        $loader->fallbackDirsPsr4 = ComposerStaticInit55013298d242363653752f43b77cfdfc::$fallbackDirsPsr4;
        $loader->prefixesPsr0 = ComposerStaticInit55013298d242363653752f43b77cfdfc::$prefixesPsr0;
        $loader->classMap = ComposerStaticInit55013298d242363653752f43b77cfdfc::$classMap;

    }, null, ClassLoader::class);
}
```
这里的工作，就是把从各个地方搜刮到的映射数组，绑定到$loader的变量中，简单的说，就是赋值，为什么赋值要这么写？因为需要赋值的是私有变量，众所周知，private在外头无法调用，所以这里要使用 Closure::bind 匿名函数的绑定功能<s>搞得这么麻烦，用public它不香吗( ꒪⌓꒪)</s>，这个东西是怎么使用的，可以参考这篇文章，
> [PHP中闭包Closure::bind详解](https://blog.csdn.net/qq_27718961/article/details/91043221)

总而言之，这里就是单纯的赋值。

接下来我们回到autoload_real.php
```php
$loader->register(true);
```
注册autoload方法。  
此方法即composer autoload的核心，有了它，就像土地有了金坷垃。(•‾̑⌣‾̑•)✧˖°  
那么接下来的几个关键方法，都在ClassLoader.php里，我们一一学习。
```php
public function register($prepend = false)
{
    spl_autoload_register(array($this, 'loadClass'), true, $prepend);
}
```
略。

```php
public function loadClass($class)
{
    if ($file = $this->findFile($class)) {
        includeFile($file);

        return true;
    }
}
```
略。

我们看findFile方法
```php
if (isset($this->classMap[$class])) {
    return $this->classMap[$class];
}
```
这里无情地揭示了classmap的加载原理，简单粗暴，就是class与文件路径一一对应的映射，找到就是赚到。

接下来我们看findFileWithExtension方法，此方法是用来处理PSR0与PSR4标准类的加载的。
```php
$logicalPathPsr4 = strtr($class, '\\', DIRECTORY_SEPARATOR) . $ext;

$first = $class[0];
if (isset($this->prefixLengthsPsr4[$first])) {
    $subPath = $class;
    while (false !== $lastPos = strrpos($subPath, '\\')) {
        $subPath = substr($subPath, 0, $lastPos);
        $search = $subPath . '\\';
        if (isset($this->prefixDirsPsr4[$search])) {
            $pathEnd = DIRECTORY_SEPARATOR . substr($logicalPathPsr4, $lastPos + 1);
            foreach ($this->prefixDirsPsr4[$search] as $dir) {
                if (file_exists($file = $dir . $pathEnd)) {
                    return $file;
                }
            }
        }
    }
}
```
以上即是PSR4标准类的自动加载代码，简单说一下自己的理解。  
先通过首字母索引在prefixLengthsPsr4数组中初步确定类的存在，之后在prefixDirsPsr4数组中通过命名空间一层一层向上查找，直到找到匹配的顶层命名空间，之后遍历此顶层命名空间，匹配到符合规则的类文件并返回完整路径。

关于在此代码中的一些疑问：
```php
if (isset($this->prefixLengthsPsr4[$first]))
```
为什么要有这个if，似乎不使用也一样能运行？  
个人理解是因为增加这个判断相当于使用了索引，可以提高效率。

prefixLengthsPsr4数组中保存的是命名空间的长度，有何用途？  
在参考其他关于composer autoload的文章时有看到这部分代码是会使用到这个值的，用来替换路径，但是我目前这个代码并没有使用到，所以可能是版本不同，代码略微有变化，但基本原理差不多。

以上的代码完成了classmap，PSR0及PSR4三种方式的autoload，那么回到autoload_real.php，我们看看files方式是如何autoload的（如果有的话）。
```php
if ($useStaticLoader) {
    $includeFiles = Composer\Autoload\ComposerStaticInit55013298d242363653752f43b77cfdfc::$files;
} else {
    $includeFiles = require __DIR__ . '/autoload_files.php';
}
foreach ($includeFiles as $fileIdentifier => $file) {
    composerRequire55013298d242363653752f43b77cfdfc($fileIdentifier, $file);
}
```
呃，简单粗暴，遍历files数组，一次性预加载进来。  
这种方式通常是写一些全局函数的，方便项目进行调用。

完。

### 参考资料
>[深入解析 composer 的自动加载原理](https://segmentfault.com/a/1190000014948542)

>[PHP中闭包Closure::bind详解](https://blog.csdn.net/qq_27718961/article/details/91043221)
