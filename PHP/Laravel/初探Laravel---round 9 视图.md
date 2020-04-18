# 初探Laravel---round 9 视图

<!-- more -->

*Mission Start!*

## 视图的基本用法

Laravel中，可以通过两种方式返回视图，一种是直接以字符串形式返回视图源码，另一种是返回视图文件。    

① 第一种方法就是直接使用return返回字符串：

```php
public function index()
{
    return '<html></html>';
}
```

② 第二种是通过view()返回视图文件，视图存放在laravel/resources/views目录下。view函数是定义的全局函数相当于View::make()方法，实质调用的是Illuminate\View\Factory的make方法，进而创建Illuminate\View\View类的实例化对象。

```php
public function index()
{
    return view('index');
}
```

以上用例会加载laravel/resources/views/index.blade.php文件，如果视图文件存在laravel/resources/views下的子目录下，例如：视图文件为laravel/resources/views/user/login.blade.php，则：

```php
public function index()
{
    return view('user.login');
    // 或
    // return view('user/login');
}
```

## 数据传递

如果需要传递数据到视图中，有三种方法：

> ① 通过数组的形式，如：```return view('index', ['username' => 'Alan', 'age' => 18]);```
> 
> ② 通过with函数，如：```return view('index')->with('username', 'Alan')->with('age', 18);```
> 
> ③ 通过with加变量名的形式，如：```return view('index')->withUsername('Alan')->withAge(18);```

## blade模板

Blade模板是Laravel提供的视图文件引擎，该模板可以通过模板继承和区块实现高度的代码复用和清晰的视图结构。blade模板文件以```.blade.php```为后缀。

### blade模板结构布局

```php
// layout.blade.php
<html>
    <head></head>
    <body>
        @section('navbar')
            This is the layout navbar.
        @show
        
        @yield('content')
        <footer>
            <p>This is footer.</p>
        </footer>
    </body>
</html>

// view.blade.php
@extend('layout')

@section('navbar')
    <p>This is the view navbar.</p>
    @parent
@endsection

@section('content')
    <div>
        This is the content.
        include('sidebar')
    </div>
@endsection

// sidebar.blade.php
<div>
    This is the sidebar.
</div

```

使用```return view('view');```即可使用视图，显示内容为：

```php
This is the view navbar.
This is the layout navbar.
This is the content.
This is the sidebar.
This is footer.
```

以下为blade模板中用于布局的语法标签：

> @extends('布局文件名')：用于继承一个布局文件
> 
> @parent：用于显示继承的布局模板中的内容
> 
> @section('区块名')：定义一个区块，其可以有不同的结尾标：@show用于显示区块；@stop和@endsection用于结束这个区块；@overwrite用于重写前面的区块。如果在布局模板文件中用@stop或@endsection结束这个区块，则视图文件将无法覆盖这个区块。
> 
> @yield('区块文件', '默认内容')：用于布局文件中显示一个区块。
> 
> @include('子视图名称')：加载子视图文件

### blade模板的过程控制语法

**blade模板变量输出**：

```php
{{ $name }}
// 效果对应的PHP原生代码
<?php echo $name; ?>
```

**三目运算符```?:```的效果**：

```php
{{ $name or 'default' }}
// 效果对应的PHP原生代码
<?php echo isset($name) ? $name : 'default'; ?>
```

**禁止花括号被解析**：

```php
@{{ $name }} // 输出{{ $name }}}
```

**条件控制**：

```php
@if($age == 18)
    i am {{$name}}
@elseif($age == 19)
    i am {{$age}}
@else
    i am {{$name}}, {{$age}}
@endif
```

**循环控制**：

```php
// foreach
@foreach ($data as $key => $value)
    <p>{{$key}} => {{$value}}</p>
@endforeach

// for
@for ($i = 0; $i < count($data); $i++)
    {{$i}} => {{$data[$i]}}
@endfor

// while
@while (true)
    While loop
@endwhile
```

其中在foreach循环中，可以使用```$loop```变量获取相应的循环信息。该变量包含的信息有：

```html
object(stdClass)[203]
  public 'iteration' => int 1       // 第几个item，范围：1~length
  public 'index' => int 0           // 索引，范围：0~length-1
  public 'remaining' => int 3       // 倒数，范围：length-1~0
  public 'count' => int 4           // 总长度，length
  public 'first' => boolean true    // 是否为第一个
  public 'last' => boolean false    // 是否为最后一个
  public 'depth' => int 1           // 当前循环嵌套层级
  public 'parent' => null           // 父级循环的loop变量信息
```

### 注释

```php
{{-- 注释 --}}
```

*Mission Complete!*