# PHP扩展之全局变量的使用

使用C语言开发PHP扩展时可能需要使用到全局变量，在使用全局变量时涉及到线程安全的问题，PHP设计了TSRM(线程安全资源管理器)来解决这个问题。在扩展开发中需要按照TSRM的规范定义全局变量。

<!-- more -->

## ZEND_BEGIN_MODULE_GLOBALS、ZEND_END_MODULE_GLOBALS

PHP为扩展的全局变量提供一种存储方式：每个扩展将自己所有的全局变量统一定义在结构体中，然后将这个结构体注册到TSRM中。

ZEND_BEGIN_MODULE_GLOBALS、ZEND_END_MODULE_GLOBALS便是用来定义这个结构体的。使用方式为：

```c
ZEND_BEGIN_MODULE_GLOBALS(extension_name)
    zend_long   val
    HashTable   table
ZEND_END_MODULE_GLOBALS(extension_name)
```

## ZEND_DECLARE_MODULE_GLOBALS

定义好存储全局变量的结构体后，便需要将该结构体注册进TSRM中。ZEND_DECLARE_MODULE_GLOBALS宏就是用于处理这个操作。

```c
ZEND_DECLARE_MODULE_GLOBALS(extension_name)
```

## ZEND_MODULE_GLOBALS_ACCESSOR

访问扩展中定义的全局变量：

```c
ZEND_MODULE_GLOBALS_ACCESSOR(extension_name, val)
```

当然你也可以自己定义一个宏进行访问：

```c
#define TEST_G(val) ZEND_MODULE_GLOBALS_ACCESSOR(extension_name, val)
```

> 需要注意的是，在一个PHP扩展中并不是只能定义一个全局变量结构体，数量是不限制的。
