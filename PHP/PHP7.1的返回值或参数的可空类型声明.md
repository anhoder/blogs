# PHP7.1的返回值或参数的可空类型声明

<!-- more -->

在PHP7中加入了参数数据类型声明以及返回值数据类型声明，例如：

```php
function test1(string $name) : string
{
    return "Hello" . $name;
}  
```

但很多时候我们都会遇到返回值类型或者参数类型为null，即Nullable Types（可空类型），例如：

```php
function test2(string $name) : string
{
    if ($name == null) "name is null";
    else if ($name == "Alan") return null;
    else return "Hello" . $name;
}
```

但是上面这种写法是不正确的，因为在某些情况下会报错，例如：

> ① 当$name = null时， 报错：Fatal error: Uncaught TypeError: Argument 1 passed to test2() must be of the type string, null given（参数必须为字符串类型，不应该 null）
>
> ② 当$name = 'Alan'时，报错：Uncaught TypeError: Return value of test2() must be of the type string, null returned（返回值必须为字符串，而不应该是null）

当遇到可空数据类型时，我们可以这么写：

```php
function test2(?string $name) : ?string
{
    if ($name == null) "name is null";
    else if ($name == "Alan") return null;
    else return "Hello" . $name;
}
```

<p style="color:red;font-size:20px;">注意：PHP 7.1版本中才支持Nullable Type</p>