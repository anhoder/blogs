# PHP的浅复制和深复制

<!-- more -->

*Mission Start!*

很早就接触到了这个东西，只是没有系统的去了解，写个博客加深记忆 :)。

## 一、概念
**浅复制**：引用赋值，复制的变量和被复制的变量指向同一个位置，对其中一个进行修改会导致从另一个变量访问的值也发生改变；    
    
**深复制**：完全复制，两个变量相互独立，不产生影响。   
   
## 二、详细区别
### 1、情况一 
最常见的就是**普通变量**和**对象**之间的区别：   
> 普通变量赋值时是深拷贝，具体采用的是*引用计数+写时复制*，因此在进行值的修改时两个变量之间是互不影响的。   
> 而对于对象来说，则是使用引用赋值（赋值、作为函数参数传递），即浅复制，两个变量之间会相互影响、相互羁绊😏。  
 
举个栗子🌰：   

```php
// 普通变量
$a = '原值';
$b = $a;        // 此时，$a和$b的值都为'原值'
$b = '修改的值'; // $a为'原值'，$b为'修改的值'

// 对象
class C
{
    public $var = '原值';
}
$c = new C();   // $c->var的值为'原值'
$d = $c;
$d->var = '修改的值';   // 此时$c->var为'修改的值'，$d->var也为'修改的值'
```

那如果想要对对象进行深拷贝应该如何做呢？在PHP中提供了**clone**可以实现。如：

```php
class A
{
    public $var = '原值';
}
$a = new A();
$b = clone $a;
$b->var = '修改的值';
echo $a->var, PHP_EOL;  // 原值
echo $b->var, PHP_EOL;  // 修改的值
```
### 2、情况二
用上述方法很多情况下可以进行浅赋值，但如果被复制对象中存在对象属性的话，原对象的普通对象会被深复制，而原对象的对象属性还是进行浅复制（引用赋值）。例如：

```php
class A
{
    public $var_a = 'class A';
}
class B
{
    public $var_b = 'class B';
    public $var_b_obj;
    
    public function __construct()
    {
        $this->var_b_obj = new A();
    }
}
$one = new B();
$two = clone $one;
// $one->var_b和$two->var_b都为'classB'
// $one->var_b_obj->var_a和$two->var_b_obj->var_a都为'classA'
$two->var_b = 'new class B';
$two->var_b_obj->var_a = 'new class A'; 
// 至此，$one->var_b='class B'，$two->var_b='new class B'
// $one->var_b_obj->var_a和$two->var_b_obj->var_a都为'new class A'
```
    
对于这种办法如何实现深复制呢？

#### 方法一
使用__clone魔术方法进行复制。如：

```php
class A{}
class B
{
    public $obj;
    
    public function __construct()
    {
        $this->obj = new A();
    }
    
    public function __clone()
    {
        $this->obj = clone $this->obj;
    }
}
```
#### 方法二
使用序列化(serialize)和反序列化(unserialize)实现。如：

```php
class A {}
class B
{
    public $a;
    
    public function __construct()
    {
        $this->a = new A();
    }
}
$one = new A();
$temp = serialize($one);
$two = unserialize($temp); // 此时的$two与$one是不会影响的两个变量
```

> **注意：
> ① 使用序列化和反序列化时，将会触发__sleep和__wakeup魔术方法
> ② 同样的原理我们也可以使用json相关函数进行对象复制，若使用json_encode和json_decode将丢失原对象中的方法**

*Mission Complete!*


