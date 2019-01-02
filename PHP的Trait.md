# PHP的Trait

*Mission Start!*

PHP是一种单继承语言，类似的还有Ruby等，与C++等多继承语言相比，代码复用性需要使用另外的方法去解决。在Ruby中使用Mixin混入类来解决。而在PHP中，使用Trait来实现。    

Trait和类相似，但不能像类一样进行实例化，而是需要使用use关键字添加到其他类的内部。    

## Trait的简单使用：

```php
trait Hello
{
    public function hello()
    {
        echo 'hello() from Trait Hello';
    }
}

class A
{
    use Hello;
    
}

$a = new A();
$a->hello(); // hello() from Trait Hello
```

## Trait的重要性质

1. 优先级：当前类的方法会覆盖trait中的方法，trait中的方法会覆盖基类的方法。
2. 多个Trait组合：use后跟多个trait并通过逗号分隔；或者使用use一一列出trait。
3. 冲突解决：如果两个trait都插入了一个同名的方法，若没有明确解决冲突，会产生一个致命错误。为了解决多个trait在同一类中的命名冲突，**需要使用insteadof操作符来明确指定使用冲突方法中的哪一个**。同时，**可以通过as操作符将其中一个冲突的方法以另外一个名称引入进来**。
4. 修改方法的访问控制：使用as语法可以用来调整方法的访问控制。
5. trait的抽象方法：在trait中可以使用抽象成员，使得类中必须实现这个抽象方法。
6. trait中可以使用静态方法和静态变量。
7. 在trait中同样可以定义属性。


*Mission Complete!*