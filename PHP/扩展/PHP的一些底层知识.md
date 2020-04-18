# PHP的一些底层知识

自己在学习PHP底层时的一些笔记。

<!-- more -->

> *阅读[https://github.com/huanghantao/study-note/blob/master/PHP/PHP%E6%89%A9%E5%B1%95%E5%BC%80%E5%8F%91%E7%AC%94%E8%AE%B0.md](https://github.com/huanghantao/study-note/blob/master/PHP/PHP%E6%89%A9%E5%B1%95%E5%BC%80%E5%8F%91%E7%AC%94%E8%AE%B0.md)的笔记*

## 1. php-config

`php-config`是一个简单的命令行脚本，**用于获取PHP的安装及配置信息**。当安装多个PHP版本时，可以使用`--with-php-config`来指定使用哪个版本的PHP来进行编译。

## 2. PHP的扩展类型

PHP的扩展可分为静态扩展和动态扩展两类。

* 静态扩展是与PHP源码一起进行编译安装的，编译PHP时通过指定参数`--enable`和`--with`来启用扩展。
* 动态扩展是直接编译扩展的源代码，生成`.so`(Linux)或`.dll`(Windows)的动态链接库，然后在php.ini配置文件中开启。

> 在静态编译PHP时，enable 是启用 PHP 源码包自带，但是默认不启用的扩展，比如 ftp 和 exif 扩展。with 是指定扩展依赖的资源库的位置，如果是默认位置，就可以留空。

## 3. PHP函数的底层实现

PHP的函数分为两种，一种为内置函数`zend_internal_function`，另一种为用户函数`zend_user_function`。

* 内置函数在PHP的内核实现，通过c/c++进行编写
* 用户函数是用户使用PHP自定义的函数，这种函数会被ZE(Zend Engine)编译为opcode来执行

PHP在编译阶段将用户自定义的函数编译为独立的opcodes,保存在EG(function_table)中,调用时重新分配新的zend_execute_data(相当于运行栈),然后执行函数的opcodes,调用完再还原到旧的zend_execute_data，即原来的函数，继续执行。

zend_function的结构中的op_array存储了该函数中所有的操作,当函数被调用时,ZE就会将这个op_array中的opline一条条顺次执行, 并将最后的返回值返回。 从VLD扩展中查看的关于函数的信息可以看出,函数的定义和执行是分开的,一个函数可以作为一个独立的运行单元而存在。

## 4. PHP的线程安全

配置编译PHP时加参数`--enable-maintainer-zts`则编译出的php为Zend线程安全（ZTS），否则不是线程安全（NTS）。

当**使用pthread（POSIX threads）扩展时，或者当web服务器为Apache2 mpm-worker或IIS**使用PHP作为模块时，请考虑使用ZTS。当**使用FastCGI / FPM或Apache2 mpm-prefork**时，您不需要ZTS，因为在PHP运行时使用的多进程处理。

NTS是运行PHP的首选方式。NTS还使您更容易编写和调试扩展。

## 5. Zend引擎的执行过程

> 1. Zend Engine调用词法分析器将PHP脚本分割为一个一个的token
> 2. Zend Engine会将得到的token提交给语法分析器，生成抽象语法树(Abstract Syntax Tree)
> 3. ZE调用zend_compile_top_stmt()函数将抽象语法树解析为一个一个的opcode，opcode一般会以op_array的形式存在，它是PHP执行的中间语言
> 4. 最后,ZE调用zend_executor来执行op_array，输出结果

### Token

PHP提供`token_get_all`函数来获取token，例如：

```php
$token = token_get_all('<?php echo \'hello, world\';');
var_dump($token);
```

输出结果为：

```sh
array(5) {
  [0] =>
  array(3) {
    [0] =>
    int(379)
    [1] =>
    string(6) "<?php "
    [2] =>
    int(1)
  }
  [1] =>
  array(3) {
    [0] =>
    int(328)
    [1] =>
    string(4) "echo"
    [2] =>
    int(1)
  }
  [2] =>
  array(3) {
    [0] =>
    int(382)
    [1] =>
    string(1) " "
    [2] =>
    int(1)
  }
  [3] =>
  array(3) {
    [0] =>
    int(323)
    [1] =>
    string(14) "'hello, world'"
    [2] =>
    int(1)
  }
  [4] =>
  string(1) ";"
}
```

### AST抽象语法树

以下工具可以用于查看AST结构信息：

* https://pecl.php.net/package/ast  (扩展)
* https://dooakitestapp.herokuapp.com/phpast/webapp/ (在线)
* https://github.com/nikic/PHP-Parser (PHP解析工具)


### Opcode

opcode是Zend虚拟机可识别的指令，php7共有173个opcode，定义在zend_vm_opcodes.h中，这些中间代码会被Zend VM(Zend虚拟机)直接执行。

查看opcode可以使用：

* https://pecl.php.net/package/vld (扩展)
* https://3v4l.org/UBstu/vld#output (在线)

## 6. EG变量

executor_globals是一个全局变量，存储着许多信息(当前上下文、符号表、函数/类/常量表、堆栈等)，EG宏就是用于访问executor_globals的某个成员
