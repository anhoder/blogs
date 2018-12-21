# 读《细说PHP》

[TOC]

*Mission Start!*

## PHP的错误ERROR与异常EXCEPTION

*注意：在不同的语言中，错误与异常的定义不同。因此以下内容只在PHP中成立。*

### 错误ERROR

错误：属于PHP脚本自身的问题，可能是由错误的语法、服务器环境等导致，使得编译器无法通过检查，甚至无法运行的情况。    

**错误不能被try/catch捕获**    

错误的级别：

 * Fatal Error  致命错误，脚本终止运行
    * E_ERRO: 致命的运行时错误，阻止脚本的执行
    * E_CORE_ERROR: PHP初始化启动过程发生的错误，由PHP引擎核心产生
    * E_COMPILE_ERROR: 致命编译时错误，由Zend脚本引擎产生
    * E_USER_ERROR: 用户产生的错误信息，在代码中使用函数trigger_error()产生（错误类型设置为E_USER_ERROR）
 * Parse Error  编译时解析错误，语法错误，脚本终止运行
    * E_PARSE: 编译时语法解析错误，仅有分析器产生
 * Warning Error  警告错误，脚本不终止运行
    * E_WARNING: 运行时警告，非致命错误
    * E_CORE_WARNING: PHP初始化启动过程中发生的警告，非致命错误，由PHP引擎核心产生
    * E_COMPILE_WARNING: 编译时警告，非致命错误，由Zend脚本产生
    * E_USER_WARNING: 用户产生的警告信息，在代码中使用函数trigger_error()产生（错误类型设置为E_USER_WARNING）
 * Notice Error  通知错误，脚本不终止运行
    * E_NOTICE: 运行时通知，表示脚本遇到可能会表现为错误的情况，但是在可以正常运行的脚本里面也可能会有类似的通知
    * E_USER_NOTICE: 用户产生的通知信息，在代码中使用函数trigger_error()产生（错误类型设置为E_USER_NOTICE）

    另外：
    
    * E_ALL: 除E_ALL外的所有的错误、警告和注意
    * E_STRICT: 关于PHP版本移植的兼容性和互操作性建议

### 异常EXCEPTION

异常：在程序运行期间出现的不符合预期的情况，允许发生但不正常的情况。属于业务逻辑的错误，而不是语法或编译的错误。

### 捕获错误和异常

在PHP中，用于捕获异常和错误的三个重要函数：**error_reporting**、**set_error_handler**、**register_shutdown_function**、**error_get_last**、**set_exception_handler**。

#### error_reporting函数

设置报告哪些错误

```php
int error_reporting([int $err_type])

/**
 * int $err_type 可选参数，需要设置错误等级
 * 返回值：int 返回旧的错误级别
 */
```

用例：

```php
error_report(E_ALL | E_STRICT);
```

#### set_error_handler函数

捕获错误，并执行用户自定义的错误处理函数。

```php
mixed set_error_handler(callable $error_handler 
    [, int $error_types = E_ALL | E_STRICT])
/**
 * callable $error_handler
 *     回调函数，需要声明四个参数：错误类型、错误信息、错误所在文件、错误所在行。
 *     第五个参数可选，指向错误发生时活动符号表的array，但在PHP 7.2.0被废）
 *     
 * int $error_types
 *     可选，规定在哪个错误报告级别显示用户定义的错误
 */
```

注意事项：

> 1. 如果存在该方法，相应的error_reporting()就不能再使用，所有错误都会交给自定义函数处理；
> 2. 此函数只能捕获系统产生的一些Warning、Notice级别的错误，不能捕获：E_ERROR、E_PARSE、E_CORE_ERROR、E_CORE_WARNING、E_COMPILE_WARNING以及set_error_handler()函数所在文件中产生的E_STRICT。

用例：

```php
<?php
    set_error_handler('error_handler');
    function error_handler($err_type, $err_msg, $err_file, $err_line)
    {
        var_dump($err_type);
        var_dump($err_msg);
        var_dump($err_file);
        var_dump($err_line);
    }
    
    // 或者
    
    set_error_handler(array('class_name', 'function_name'));
```

#### register_shutdown_function函数

该函数时PHP脚本执行结束前最后一个调用的函数。通过这个函数就可以在脚本结束前判断是否产生错误。这是需要使用到**error_get_last**函数，获取本次执行产生的所有错误。**register_shutdown_function可以配合set_error_handler捕获所有错误**

```php
void register_shutdown_function(callable $callback
    [, mixed $parameter
    [, mixed $...]])
/**
 * callable $callback 回调函数
 * mixed $parameter 传递给回调的参数
 * ... 更多回调
 */
```

注意事项：

> 1. 该函数可以捕获的错误：Fatal Error、Parse Error等
> 2. register_shutdown_function所在的文件发生语法错误，不能触发回调函数（用例之后会说到）

用例：

```php
<?php
    register_shutdown_function('shutdown_handler');
    function shutdown_handler()
    {
        if ($err = error_get_last()) {
            var_dump($err);
        }
    }
```

但是，如果执行下列测试文件，是不能触发调用回调函数：

```php
<?php
    register_shutdown_function('shutdown_handler');
    function shutdown_handler()
    {
        if ($err = error_get_last()) {
            var_dump($err);
        }
    }
    echo 1++;
```

为什么不能触发呢？这是因为：

> 如果register_shutdown_function所在文件发生语法错误，该脚本不能通过PHP的语法分析，register_shutdown_function函数就不会被执行，也就没有注册回调函数。
> 
> 因此，我们需要保证register_shutdown_function所在文件没有语法错误。

将上述代码修改为：

```php
<?php
    register_shutdown_function('shutdown_handler');
    function shutdown_handler()
    {
        if ($err = error_get_last()) {
            var_dump($err);
        }
    }
    require 'a.php';
    
// a.php文件
<?php
    echo 1++;
```

修改之后，因为register_shutdown_function所在文件没有语法错误，通过了PHP的语法分析，所以register_shutdown_function函数得以正常运行，便能注册回调函数了。    

**因此，最好将register_shutdown_function单独放在一个的文件中，并保证该文件没有语法错误。**

#### error_get_last函数

该函数用于获取本次执行中产生的错误信息。

```php
array error_get_last()
/**
 * 返回值：array 
 * type =>　错误类型
 * message => 错误信息
 * file => 发生错误的文件
 * line => 发生错误的行号
 */
```

#### set_exception_handler函数

设置异常处理函数，如果异常没被捕获就会进到该方法中，并且在回调函数调用后异常会中止。

```php
callable set_exception_handler(callable $exception_handler)

/**
 * callable $exception_handler 回调函数，需要接受一个参数：捕获的异常对象
 * 返回值： callable 返回定义的回调函数的名称
 */
```

