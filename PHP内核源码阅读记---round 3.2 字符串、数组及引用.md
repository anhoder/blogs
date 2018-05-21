# PHP内核源码阅读记---round 3.2 字符串、数组及引用

*Mission Start!*

前面了解到了PHP变量的实现方式和变量类型，接下来介绍一下字符串、数组及引用
## 一、字符串
PHP中并不是使用C语言的char来表示字符串，而是定义了一个结构体zend_string，前面提到变量的值是保存在zval结构体里的zend_value中的。在表示字符串时，zend_value的通过zend_string指针指向具体的zend_string结构体。zend_string结构如下：

```c
typedef struct _zend_string zend_string;
struct _zend_string {
    zend_refcounted_h gc;
    zend_ulong h;
    size_t len;
    char val[1];
};
```
> * gc: 变量的引用计数信息，用于内存管理;
> * h: 字符串通过Times 33算法计算得到的Hash Code;
> * len: 字符串长度；
> * val: 字符串内容。

val[1]并不是表示它只能存储一个字节（长度为1是保存结束符"\\0"），因为val是变长数组，在字符串分配时实际是类似这样的操作： malloc(sizeof(zend_string) + 字符串长度)，
也就是会多分配一些内存，而多出的这块内存的起始位置就是val，这样就可以直接将字符串内容存储到val中，通过val进行读取。若val使用char*指针，则需要额外分配一次内存，而变长结构体不仅可以省一次内存分配，而且有助于内存管理。另外，val多出来的一个字节用于存储字符串的最后一个字符"\\0"。

## 二、数组

在PHP中，数组是一种很强大、很灵活的数据结构，它的底层实现是哈希表（HashTable）。哈希表结构为： 

```c
struct _zend_array {
    zend_refcounted_h gc;
    // 这个union可以先忽略
    union {
        ...
    } u;
    // 用于散列函数映射存储元素在arData数组中的下标
    uint32_t nTableMask;
    // 存储数组元素，每个元素的结构统一为Bucket，指向第一个Bucket
    Bucket *arData;
    // 已用Bucket数
    uint32_t nNumUsed;
    // 数组实际存储的元素数
    uint32_t nNumOfElements;
    // 数组的总容量
    uint32_t nTableSize;
    uint32_t nInternalPointer;
    // 下一个可用的数值索引，如arr[] = 1; arr['a'] = 2; arr[] = 3;，则nNextFreeElement = 2
    zend_long nNextFreeElement;
    dtor_func_t pDestructor;
};
```

### 哈希表（散列表，HashTable）的基本实现
哈希表是根据关键码值直接进行访问的数据结构，它是直接通过键key映射到内存地址上去，从而加快查找速度。哈希表主要由两部分组成：存储元素数组、散列函数。散列函数的作用是根据key映射出元素的存储位置，通常用取模作为散列函数。

> 如果直接用散列函数的输出值作为该元素在存储元素数组中的下标的方式存在一个问题：元素在数组中的分布是随机的、无序的。但其实在PHP中数组元素的位置是有序的，数组中各元素的顺序与其插入顺序一致。

PHP中是如何实现的呢？

> 为了实现散列表的有序性，PHP中的散列表在散列函数与元素数组之间加一层**映射表**，这个映射也是一个数组，大小与存储元素的数组相同，存储了元素在实际存储数组中的下标。**元素按照先后顺序一次插入实际存储数组，然后将其数组下标按照散列函数的位置再新加到映射表中。**

### 哈希冲突
两个不同的key值经过散列函数散列后得到的哈希值可能会相同，但一个位置只能存储一个元素，这种冲突就叫做哈希冲突。

**解决方法**：如果可以把哈希值相同的元素保存起来就可以解决了，常见的解决办法就是把冲突的元素串成链表，原来的位置保存链表的头指针，查找时需要遍历这个链表，逐个比较key，从而找到目标元素。


## 三、引用

引用类似于C语言中指针的概念。在PHP中，使用 & 符生成一个引用变量。在执行时，首先分配一个zend_reference结构，这个结构就是引用类型的结构体，其内嵌了一个zval，这个zval指向被引用的zval的value，然后将原zval的类型修改为IS_REFERENCE，原zval的value指向新创建的zend_reference结构。也就是说。& 将变量的数据类型转换为引用，而新生成的引用结构指向原来的value。zend_reference结构如下：

```c
struct _zend_reference {
    zend_refcounted_h gc;
    zval val;
};
```

例如：
> ① 定义\$a

> ```php
$a = '123456';
```

> PHP内部：
> ![PHP引用1](https://media.alan123.xyz/imgs/blogs/phpreference/1.jpg)

> ② 定义\$b为\$a的引用

> ```php
$b = &$a;
```

> PHP内部：
> ![PHP引用2](https://media.alan123.xyz/imgs/blogs/phpreference/2.jpg)

*Mission Complete!*




