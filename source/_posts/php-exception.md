---
title: php异常处理的深入
date: 2018-10-27
tags:
- php
categories:
- 深入
comments: true
---

## 引出

如果你调一个类，调用时数据验证时报了个错，你会以什么方式返回

数组，布尔值？

数组这个可以带错误原因回来，那布尔值呢？

返回了个 false, 报错时把错误放在类变量里？
还是专门用一个获取错误的方法进行获取？

上面说的情况是代码完全没有问题的情况。
那如果是一些第三方的工具包，你又怎么知道他里面的执行会不会导致整个系统崩溃。

你说本地运行是没问题的，环境这种东西不好说。

所以我们就用到了 异常 这个东西

下面是我们需要了解的问题

什么时候抛异常？怎么接异常？异常要怎么处理？他的使用场景又是什么？

## 基础知识

1. 基础操作

    try ... catch()
    throw

2. [错误级别](http://php.net/manual/zh/errorfunc.constants.php)

    致命错误 E_ERROR，
    语法错误 E_PARSE，
    警告错误 E_WARNING，
    通知错误 E_NOTICE

3. php异常处理类

[预定义异常](http://php.net/manual/zh/reserved.exceptions.php)

```txt
    * ErrorException (extends Exception)
```

[SPL异常类](http://php.net/manual/zh/spl.exceptions.php)

```txt
    * LogicException (extends Exception)      // 表示程序逻辑中的错误的异常。这种异常应该直接在代码中的修复
        * BadFunctionCallException            // 回调调用未定义的函数或缺少一些参数时会抛出该异常
            * BadMethodCallException          // 回调方法是一个未定义的方法或缺失一些参数时会抛出该异常
        * DomainException                     // 值不遵守定义的有效数据域时会抛出该异常
        * InvalidArgumentException            // 参数不是预期类型时会抛出该异常
        * LengthException                     // 长度无效时会抛出该异常
        * OutOfRangeException                 // 请求非法索引时引发的异常，这应该在编译时就检测到的错误

    * RuntimeException (extends Exception)    // 在运行时发生的错误会抛出该异常
        * OutOfBoundsException                // 值不是有效键时会抛出该异常，这表示在编译时无法检测到的错误
        * OverflowException                   // 在向完整容器中添加元素时引发的异常
        * RangeException                      // 在程序执行期间为指示范围错误而引发的异常。通常这意味着除了/overflow以外还有一个算术错误。这是运行时的DomainException版本
        * UnderflowException                  // 在空容器上执行无效操作(如删除元素)时引发的异常
        * UnexpectedValueException            // 值与一组值不匹配时会抛出该异常。通常，当一个函数调用另一个函数并期望返回值为某种类型或值(不包括算术或缓冲区相关错误)时，就会发生这种情况
```

4. 异常处理相关函数

```txt
    error_reporting // 设置报告的错误级别
    register_shutdown_function // 注册一个会在php中止时执行的函数
    set_error_handler // 设置用户自定义的错误处理函数
    set_exception_handler // 设置用户自定义的异常处理函数
    error_get_last // 获取最后发生的错误
```

## 使用场景

1. 系统

    主要抓的是无法预测的错误，统一返回，没有使用 try...catch 接收的异常直接跳进设置的方法中

```php
<?php

namespace App\Exception;

use Exception;

/**
 * 异常句柄（入口）类
 */
class Handler
{
    // 默认错误处理
    public static function errorHandler($errno, $errstr, $errfile = '', $errline = 0)
    {
    }

    // 默认异常处理
    public static function exceptionHandler($ex)
    {
        try {
            throw $ex;
        } catch (Order $e) {
            echo "订单异常";
        } catch (Goods $e) {
            echo "商品异常";
        } catch (User $e) {
            echo "用户异常";
        } catch (Exception $e) {
            echo "其他异常";
        }
    }

    // 致命错误处理
    public static function fatalErrorHandler()
    {
        if ($e = error_get_last()) {
            print_r($e);
        }
    }
}

/**
 * 订单异常
 */
class Order extends Exception
{
}

/**
 * 商品异常
 */
class Goods extends Exception
{
}

/**
 * 用户异常
 */
class User extends Exception
{
}

```

```php
<?php
// 入口文件中

error_reporting(E_ERROR | E_WARNING | E_PARSE | E_NOTICE);

register_shutdown_function(array('App\\Exception\\Handler', 'fatalErrorHandler'));

set_error_handler(array('App\\Exception\\Handler', 'errorHandler'));

set_exception_handler(array('App\\Exception\\Handler', 'exceptionHandler'));


```

2. 工具

    定义自定义的异常，一有错误直接抛出。使用工具的程序只需通过 Exception 接收异常即可, 所有异常都通过这个进行处理的

```php
<?php

namespace Testlin\Db\Exception;

use Exception;

interface ExceptionInterface
{
}

class Db extends Exception implements ExceptionInterface
{
}

class Pdo extends Db
{
}
?>

<?php
namespace Testlin\Db;

use Exception;
use Testlin\Db\Exception\Pdo;

class Db
{
    protected $db;

    public function __construct($config)
    {
        $this->db = new PDO($config);

        if ($this->db == false) {
            throw new Pdo("连接失败");
        }
    }
}

?>
```

## 文章例子

1. [工具包例子](https://github.com/test-lin/pack_demo)
2. [项目例子](https://github.com/test-lin/project_demo)

## FQA

1、为什么要定自定义异常类, 系统不是已经给了很多选择，而且很多 composer 包里都只是继承一下。

答：其实自定义异常是为了用区分异常颗粒度的，比如

我定了 订单异常，商品异常，用户异常 类，但是 订单里的异常多种多样，比如订单支付异常，订单生成异常。

    * RuntimeException (extends Exception)
        * Order
            * Paymen
            * Created
        * Goods
        * User
            * Withdraw

当项目抛出异常时

```php
<?php
    try {
        $param = []; // 操作那个方法时传的参数

        throw App\Exception\Order\Payment::forParam('执行xxx操作异常', $param);
    } catch (Exception $e) {
        // 相关操作
        get_class($e); // 当前异常类 App\Exception\Order\Payment
    }
```

通过异常类名，我们可以知道是订单支付异常。这里可以代替错误号，而且更清晰明了

2、为什么有一些 composer 包里的自定义异常，有的有很多方法。有什么用处吗？

**作用1：格式化异常**

比如：抛出的异常提示是 "id=xx 的用户不存在"，我们会有以下两种写法

```php
<?php

// 普通操作
$id = 1;
throw new Payment("id={$id} 的用户不存在");

// 格式化异常
use App\Exception\Order;

class Payment extends Order
{
    public static function forId($id)
    {
        return new self(sprintf(
            'id=%s 的用户不存在',
            $id
        ));
    }
}

$id = 1;
throw Payment::forId($id);

```

**作用2：组件级别的异常**

```php
<?php

namespace Testlin\Db\Exception;

use Exception;

interface ExceptionInterface
{
}

class Mysqli extends Exception impements ExceptionInterface
{
}

class Pdo extends Exception impements ExceptionInterface
{
}


try {
    throw new Testlin\Db\Exception\Mysqli('sql 执行失败');
} catch (Testlin\Db\Exception\ExceptionInterface $e) {
    // 这里取得的异常只会是继承这个接口的异常
    // 可以只针对这个工具包进行处理
}

```