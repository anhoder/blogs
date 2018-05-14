# PHP内核源码阅读记---round 3 PHP变量的实现

*Mission Start!*

## 一、PHP变量的实现---zval结构
我们都知道在PHP中一个变量可以用来存储任意的数据类型，但实现PHP的底层C语言却是强类型语言，那么在PHP中是如何实现这样一个可以改变数据类型的变量的呢？   
   
在PHP内核中是通过一个叫zval的结构体来存储变量的，其定义在Zend/zend.h头文件中，其代码结构为：

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

## 二、变量的类型

*待续...*




