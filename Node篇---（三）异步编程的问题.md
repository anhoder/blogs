# Node篇---（三）异步编程的问题

异步编程很好用，但在很多时候也会因为与我们的思维方式不一致而产生一些问题。

## 1、异常处理的捕获
在介绍这个之前，我们需要了解Node的时间循环机制：

> 循环机制：Node进程在启动时会创建一个循环，每次循环都会检查是否有待处理事件，若存在就执行相对应的回调函数。每一次循环称为一个**Tick**。

OK，进入主题。   
在PHP或者Java中，我们常会用try、catch、finally等来捕获异常，因此，我么很有可能会写出以下的代码：

```js
try {
  someFunction(function(){
    // 回调函数的具体实现
  });
} catch (e) {
  console.log(e.getMessage());
  // ...
}
```
但实际上，在调用someFunction()方法后，回调函数会被保存起来，直至下一个Tick才会被执行，而try/catch只能捕获当前Tick内的异常。   

#### 解决方案 
Node约定将异常作为回调函数的第一个参数传回，如：

```js
someFunction(function (error, result) {
  if (error) {
    // 出现异常
  } else {
    // 正常执行
  }
});
```
如果存在异常，error为具体的异常；如果正常执行，没有异常，error则为null。

## 2、不能阻塞代码
在JavaScript中，没有提供sleep()线程沉睡功能的函数，只有setInterval()和setTimeout()这两个延时执行的函数，且不会阻塞后续代码的执行。   

#### 解决方案
当需要用到阻塞时，考虑用setTimeout去实现😂。

## 3、多线程编程
Node属于单线程的编程语言，一个Node进程不能充分利用多核CPU的资源，而复杂的业务逻辑往往要求需要很好的利用多核CPU以提供更多的计算服务。

#### 解决方案
当需要充分利用多核CPU时，可以参考Web Worker的实现，即Mater-Worker模型，通过Master创建工作线程，使用子线程去计算，以充分利用CPU资源。   

在Node中，我们可以使用Node的child_process的API去进行相关的需求实现。

## 4、}}}}}}}}嵌套问题
### 无顺序 ((A、B全部完成)->C)
TODO
### 有顺序（A->B->C）
在初使用JavaScript的时候，常常会嵌套使用以实现某些需求。   
例如：
> ```js
> $.ajax({
>   url: '/url',
>   success: function (data) {
>     // 下面的ajax请求依赖于外层ajax的返回数据data
>     $.ajax({
>       url: './url2',
>       data: data,
>       success: function (data) {},
>       error: function () {}
>     });
>   },
>   error: function (error) {}
> });
> ```
这个时候需要使用函数嵌套去实现，但相应的就会降低整个代码的可读性，而且当依赖很多时，就会出现}}}}}}}一样的多层嵌套问题，难看，不优雅。

#### 解决方案
使用Promise模式
TODO