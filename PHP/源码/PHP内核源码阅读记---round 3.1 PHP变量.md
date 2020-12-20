# PHP内核源码阅读记---round 3.1 PHP变量

<!-- more -->

*Mission Start!*

## 一、PHP变量的实现---zval结构
我们都知道在PHP中一个变量可以用来存储任意的数据类型，但实现PHP的底层C语言却是强类型语言，那么在PHP中是如何实现这样一个可以改变数据类型的变量的呢？   
   
在PHP内核中是通过一个叫zval的结构体来存储变量的，其定义在Zend/zend.h头文件中，其代码结构为：

### PHP5

```c
struct _zval_struct {
	zvalue_value value;	   /* 变量的值 */
	zend_uint refcount__gc; /* 引用计数 */
	zend_uchar type;	      /* 变量的数据类型 */
	zend_uchar is_ref__gc;  /* 是否引用 */
};
typedef struct _zval_struct zval;
```

首先看代码中的zend_uint和zend_uchar类型，它们定义在Zend/zend_types.h中，分别是unsigned int、unsigned char：

```c
typedef unsigned int zend_uint;     // zend_uint => unsigned int
typedef unsigned char zend_uchar;   // zend_uchar => unsigned char
```

而变量的值value的数据类型为zvalue_value，它是一个联合体(union，联合体详情请看第四节==>[传送门](https://blog.alan123.xyz/php/592.html))，在Zend/zend.h中定义：

```c
typedef union _zvalue_value {
	long lval;	   /* long value */
	double dval;	/* double value */
	struct {
		char *val;
		int len;
	} str;
	HashTable *ht;	/* hash table value */
	zend_object_value obj;
} zvalue_value;
```

> 1. 其中lval用于存放整型
> 2. dval用于存放浮点数据
> 3. str是一个结构体，用于存放字符串，包括字符串的值和字符串的长度
> 4. ht存放哈希表的指针（PHP中数组就是用哈希表实现的）
> 5. obj是存放对象类型的

### PHP7

```c
struct _zval_struct {
    zend_value value;   // 变量值
    union {
        struct {
            ZEND_ENDIAN_LOTH_4(
                zend_uchar type,
                zend_uchar type_flags,
                zend_uchar const_flags,
                zend_uchar reserverd)
        } v;
        uint32_t type_info;
    } u1;
    union {
        uint32_t var_flags;
        uint32_t next;
        uint32_t cache_slot;
        uint32_t lineno;
        uint32_t num_args;
        uint32_t fe_pos;
        uint32_t fe_iter_idx;
    } u2;
};
```
联合体u1：由一个结构体v和一个32位无符号整型type_info组成。ZEND_ENDIAN_LOTH_4宏用于解决字节序问题，忽略。v中定义了type用于表示value类型，type_flags是类型掩码，用于变量的内存管理。type_info实际上是将v结构的4各成员组合到一起，每个字节对应一个成员，共4个字节。
联合体u2：纯粹用于辅助功能，zval结构的value和u1共占用了12types，而系统会进行字节对齐，共占用16types，所以定义了辅助结构u2将4types进行利用。

```c
typedef union _zend_value {
    zend_long lval;             // 整型
    double dval;                // 浮点型
    zend_refcounted *counted;   // 获取不同类型的gc头部
    zend_string *str;           // 字符串
    zend_array *arr;            // 数组
    zend_object *obj;           // 对象
    zend_resource *res;         // 资源类型
    zend_reference *ref;        // 引用类型
    // 以下几个都是给内核使用的value
    zend_ast_ref *ast;          
    zval *zv;
    void *ptr;
    zend_class_entry *ce;
    zend_function *func;
    struct {
        uint32_t w1;
        uint32_t w2;
    } ww;
} zend_value;
```


## 二、变量的类型
PHP中有8大数据类型，分别为PHP中有8大数据类型，分别为**null、布尔、整型、浮点型、字符串、数组、对象、资源(resource)**。在PHP内核中是使用特定的常量对zval实例进行标识，以实现区分变量的不同类型：

| 常量 | 描述 |
| :-: | --- |
| IS_NULL | 第一次使用的变量若未被初始化，则自动被赋予IS_NULL。这个类型的值只有一个，就是NULL |
| IS_BOOL | 布尔类型的变量有两个值，true或者false。在PHP语言中，while、if等语句会自动的把表达式的值转成这个类型的。**（在PHP7中BOOL类型直接使用type类型IS_TRUE、IS_FALSE来区分true、false，而在PHP旧版本中BOOL类型也是通过整型来进行区分的）** |
| IS_LONG | PHP语言中的整型，在内核中是通过所在操作系统的signed long数据类型来表示的。在最常见的32位操作系统中，它可以存储从-2147483648 到 +2147483647范围内的任一整数。有一点需要注意的是，如果PHP语言中的整型变量超出最大值或者最小值，它并不会直接溢出，而是会被内核转换成IS_DOUBLE类型的值然后再参与计算。再者，因为使用了signed long来作为载体，所以也就解释了为什么PHP语言中的整型数据都是带符号的了。 |
| IS_DOUBLE | PHP中的浮点数据是通过C语言中的signed double型变量来存储的， 这最终取决与所在操作系统的浮点型实现。 我们做为程序猿，应该知道计算机是无法精准的表示浮点数的， 而是采用了科学计数法来保存某个精度的浮点数。 用科学计数法，计算机只用8位便可以保存2.225x10^(-308)~~1.798x10^308之间的浮点数。 用计算机来处理浮点数简直就是一场噩梦，十进制的0.5转成二进制是0.1， 0.8转换后是0.1100110011....。 但是当我们从二进制转换回来的时候，往往会发现并不能得到0.8。 我们用1除以3这个例子来解释这个现象：1/3=0.3333333333.....，它是一个无限循环小数， 但是计算机可能只能精确存储到0.333333，当我们再乘以三时， 其实计算机计算的数是0.333333*3=0.999999，而不是我们平时数学中所期盼的1.0. |
| IS_STRING | PHP中最常用的数据类型——字符串，在内存中的存储和C差不多， 就是一块能够放下这个变量所有字符的内存，并且在这个变量的zval实现里会保存着指向这块内存的指针。 与C不同的是，PHP内核还同时在zval结构里保存着这个字符串的实际长度， 这个设计使PHP可以在字符串中嵌入‘\0’字符，也使PHP的字符串是二进制安全的， 可以安全的存储二进制数据！本着艰苦朴素的作风，内核只会为字符串申请它长度+1的内存， 最后一个字节存储的是‘\0’字符，所以在不需要二进制安全操作的时候， 我们可以像通常C语言的方式那样来使用它。 |
| IS_ARRAY | 数组是一个非常特殊的数据类型，它唯一的功能就是聚集别的变量。 在C语言中，一个数组只能承载一种类型的数据，而PHP语言中的数组则灵活的多， 它可以承载任意类型的数据，这一切都是HashTable的功劳， 每个HashTable中的元素都有两部分组成：索引与值， 每个元素的值都是一个独立的zval（确切的说应该是指向某个zval的指针）。 |
| IS_OBJECT | 和数组一样，对象也是用来存储复合数据的，但是与数组不同的是， 对象还需要保存以下信息：方法、访问权限、类常量以及其它的处理逻辑。 |
| IS_RESOURCE | 有一些数据的内容可能无法直接呈现给PHP用户的， 比如与某台mysql服务器的链接，或者直接呈现出来也没有什么意义。 但用户还需要这类数据，因此PHP中提供了一种名为Resource(资源)的数据类型。 |

PHP内核提供三个宏来获取一个变量的数据类型：Z_TYPE()、Z_TYPE_P()、Z_TYPE_PP()，它们的参数分别为：zval、\*zval、\**zval。这三个宏定义在Zend/zend_operators.h中。

## 三、变量的值
PHP内核提供了三个基础宏来获取变量的值（与上述类型获取的宏参数一样）：

| 类型 | 宏 |
| :-: | --- |
| IS_BOOL | Z_BVAL、Z_BVAL_P、Z_BVAL_PP |
| IS_DOUBLE | Z_DVAL、Z_DVAL_P、Z_DVAL_PP |
| IS_STRING | 1\. 获取值：Z_STRVAL、Z_STRVAL_P、Z_STRVAL_PP 2\. 获取字符串（返回字符串地址）：Z_STRLEN、Z_STRLEN_P、Z_STRLEN_PP |
| IS_ARRAY | Z_ARRVAL、Z_ARRVAL_P、Z_ARRVAL_PP |
| IS_OBJECT | 对象是一个复杂的结构体（zend_object_value结构体），不仅存储属性的定义、属性的值，还存储着访问权限、方法等信息。 内核中定义了以下组合宏让我们方便的操作对象： OBJ_HANDLE：返回handle标识符， OBJ_HT：handle表， OBJCE：类定义， OBJPROP：HashTable的属性， OBJ_HANDLER：在OBJ_HT中操作一个特殊的handler方法。 |
| IS_RESOURCE | 资源型变量的值其实就是一个整数，可以用RESVAL组合宏来访问它，我们把它的值传给zend_fetch_resource函数，便可以得到这个资源的操作句柄。 |

## 四、变量符号表与作用域

### 变量符号表

PHP的变量符号表与zval值的映射，是通过HashTable实现的。如：

```php
<?php
$var = 'test';
```
$var的变量名会存储在变量符号表中，代表$var的类型和值的zval结构存储在哈希表中。内核通过变量符号表与zval地址的哈希映射来实现PHP变量的存取。

### 作用域

按照作用域可将PHP变量分为全局变量和局部变量，每种作用域PHP都会维护一个符号表的HashTable。当在PHP中创建一个函数或者类时，zend engine就会创建一个新的符号表，表名函数或类中的变量是局部变量，这样就实现了局部变量的保护。当用户创建一个PHP变量时，zend engine会分配一个zval，并设置相应type和初始值，并将改变量加入到当前作用域的符号表。

*Mission Complete!*



