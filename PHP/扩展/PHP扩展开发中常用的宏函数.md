# PHP扩展开发中常用的宏函数

## zval类型相关

```c

Z_TYPE(zval)        // 获取zval的类型
Z_TYPE_P(*zval)     // 获取*zval的类型
Z_TYPE_PP(**zval)   // 获取**zval的类型

/* zval类型相关常量 */
IS_NULL         // null 
IS_BOOL         // 布尔
IS_LONG         // 整型
IS_DOUBLE       // 浮点型
IS_STRING       // 字符串
IS_ARRAY        // 数组
IS_OBJECT       // 对象
IS_RESOURCE     // 资源

```

## zval值相关

```c
//操作整数的
#define Z_LVAL(zval)			(zval).value.lval
#define Z_LVAL_P(zval_p)		Z_LVAL(*zval_p)
#define Z_LVAL_PP(zval_pp)		Z_LVAL(**zval_pp)

//操作IS_BOOL布尔型的
#define Z_BVAL(zval)			((zend_bool)(zval).value.lval)
#define Z_BVAL_P(zval_p)		Z_BVAL(*zval_p)
#define Z_BVAL_PP(zval_pp)		Z_BVAL(**zval_pp)

//操作浮点数的
#define Z_DVAL(zval)			(zval).value.dval
#define Z_DVAL_P(zval_p)		Z_DVAL(*zval_p)
#define Z_DVAL_PP(zval_pp)		Z_DVAL(**zval_pp)

//操作字符串的值和长度的
#define Z_STRVAL(zval)			(zval).value.str.val
#define Z_STRVAL_P(zval_p)		Z_STRVAL(*zval_p)
#define Z_STRVAL_PP(zval_pp)		Z_STRVAL(**zval_pp)

#define Z_STRLEN(zval)			(zval).value.str.len
#define Z_STRLEN_P(zval_p)		Z_STRLEN(*zval_p)
#define Z_STRLEN_PP(zval_pp)		Z_STRLEN(**zval_pp)

#define Z_ARRVAL(zval)			(zval).value.ht
#define Z_ARRVAL_P(zval_p)		Z_ARRVAL(*zval_p)
#define Z_ARRVAL_PP(zval_pp)		Z_ARRVAL(**zval_pp)

//操作对象的
#define Z_OBJVAL(zval)			(zval).value.obj
#define Z_OBJVAL_P(zval_p)		Z_OBJVAL(*zval_p)
#define Z_OBJVAL_PP(zval_pp)		Z_OBJVAL(**zval_pp)

#define Z_OBJ_HANDLE(zval)		Z_OBJVAL(zval).handle
#define Z_OBJ_HANDLE_P(zval_p)		Z_OBJ_HANDLE(*zval_p)
#define Z_OBJ_HANDLE_PP(zval_p)		Z_OBJ_HANDLE(**zval_p)

#define Z_OBJ_HT(zval)			Z_OBJVAL(zval).handlers
#define Z_OBJ_HT_P(zval_p)		Z_OBJ_HT(*zval_p)
#define Z_OBJ_HT_PP(zval_p)		Z_OBJ_HT(**zval_p)

#define Z_OBJCE(zval)			zend_get_class_entry(&(zval) TSRMLS_CC)
#define Z_OBJCE_P(zval_p)		Z_OBJCE(*zval_p)
#define Z_OBJCE_PP(zval_pp)		Z_OBJCE(**zval_pp)

#define Z_OBJPROP(zval)			Z_OBJ_HT((zval))->get_properties(&(zval) TSRMLS_CC)
#define Z_OBJPROP_P(zval_p)		Z_OBJPROP(*zval_p)
#define Z_OBJPROP_PP(zval_pp)		Z_OBJPROP(**zval_pp)

#define Z_OBJ_HANDLER(zval, hf) 	Z_OBJ_HT((zval))->hf
#define Z_OBJ_HANDLER_P(zval_p, h)	Z_OBJ_HANDLER(*zval_p, h)
#define Z_OBJ_HANDLER_PP(zval_p, h)		Z_OBJ_HANDLER(**zval_p, h)

#define Z_OBJDEBUG(zval,is_tmp)		(Z_OBJ_HANDLER((zval),get_debug_info)?	\
						Z_OBJ_HANDLER((zval),get_debug_info)(&(zval),&is_tmp TSRMLS_CC): \
						(is_tmp=0,Z_OBJ_HANDLER((zval),get_properties)?Z_OBJPROP(zval):NULL)) 
#define Z_OBJDEBUG_P(zval_p,is_tmp)	Z_OBJDEBUG(*zval_p,is_tmp) 
#define Z_OBJDEBUG_PP(zval_pp,is_tmp)	Z_OBJDEBUG(**zval_pp,is_tmp)

//操作资源的
#define Z_RESVAL(zval)			(zval).value.lval
#define Z_RESVAL_P(zval_p)		Z_RESVAL(*zval_p)
#define Z_RESVAL_PP(zval_pp)		Z_RESVAL(**zval_pp)
```


## 创建zval

```c
ALLOC_ZVAL              // PHP 7中移除
ALLOC_INIT_ZVAL         // PHP 7中移除
MAKE_STD_ZVAL           // PHP 7中移除
// 例如：
zval *val; MAKE_STD_ZVAL(val);

ZVAL_LONG(*zval, long)      // 将整型值赋给*zval
ZVAL_NULL(*zval)            // null
ZVAL_BOOL(*zval, int)       // bool
ZVAL_TRUE(*zval)            // true
ZVAL_FALSE(*zval)           // false
ZVAL_LONG(*zval, long)      // long
ZVAL_DOUBLE(*zval, double)  // double
ZVAL_STRINGL(*zval,str,len,dup); // string with length
ZVAL_STRING(*zval, str, dup)     // string, dup指明了该字符串是否需要被复制。 值为 1 将先申请一块新内存并赋值该字符串，然后把新内存的地址复制给pzv， 为 0 时则是直接把str的地址赋值给zval。
ZVAL_RESOURCE(*zval, res);       // resource
```

## 函数相关

```c
ZEND_NUM_ARGS()             // 获取函数参数的数量

RETVAL_STRING(s)            // 函数返回字符串
RETURN_STRING(s)            // 函数返回字符串, #define RETURN_STRING(s) { RETVAL_STRING(s); return; }
RETURN_NULL()               // 函数返回null
RETURN_TRUE()               // 函数返回true
```

> 附：PHP 7中使用FAST_ZPP方式解析函数参数的宏：
  ![](media/16078683686223/16078723189095.jpg)


## 数组操作

```c
ZEND_HASH_FOREACH_VAL(ht, val)
ZEND_HASH_FOREACH_KEY(ht, h, key) 
ZEND_HASH_FOREACH_PTR(ht, ptr)
ZEND_HASH_FOREACH_NUM_KEY(ht, h) 
ZEND_HASH_FOREACH_STR_KEY(ht, key)
ZEND_HASH_FOREACH_STR_KEY_VAL(ht, key, val)
ZEND_HASH_FOREACH_KEY_VAL(ht, h, key, val)

// 例如：
ZEND_HASH_FOREACH_KEY_VAL(arr_hash, num_key, key, val) {
    //if (key) { //HASH_KEY_IS_STRING
    //}
    PHPWRITE(Z_STRVAL_P(val), Z_STRLEN_P(val));
    php_printf("\n");
}ZEND_HASH_FOREACH_END();
```

## 符号表

PHP的变量存储在符号表中，不同作用域有不同的符号表。

```c
struct _zend_executor_globals {
    ...
    HashTable symbol_table;         // 全局符号表
    HashTable *active_symbol_table; // 当前作用域符号表
    ...
};
```

```c
EG(active_symbol_table)
EG(symbol_table)
ZEND_SET_SYMBOL(symbol, var_name, val)  // 设置变量到符号表
// 例如：
ZEND_SET_SYMBOL( EG(active_symbol_table) ,  "foo" , fooval);
```

> 未完待续...