# Node篇---（四）Promise

*Mission Start!*

Promise是JavaScript社区针对异步编程提出的一套解决方案，现已被纳入ES6的标准中。   

> Promise拥有三个状态：未完成态(pending)、完成态(fulfilled)、失败态(rejected   )，其关系如下图：
> 
> ```mermaid
> graph LR;
>   A(未完成态)-->B(完成态)
>   A-->C(失败态)
> ```

## Promise的使用

```js
var promise = new Promise(function (resolve, reject) {
  if (成功) {
    resolve(result);
  } else {
    reject(error);
  }
});

promise.then((result) => {
  console.log("Success");
}, (error) => {
  console.log("Error");
});
```
上述代码中，第一段实例化Promise得到promise变量，其构造函数接受一个函数作为参数，该函数用于决定在何时调用成功或失败的回调函数。这个参数函数有两个参数：resolve、reject，分别是成功的回调函数、失败的回调函数。   
   
第二段代码中，使用Promise的成员方法then传入回调函数，其接受两个参数，分别为成功后的回调函数和失败后的回调函数（失败的回调函数为可选 ）。   
   
例如，读取一个文件：

```js
const fs = require('fs');
var promise = new Promise((resolve, reject) => {
  fs.readFile("a.txt", "utf8", function (error, data) {
    if (error) {
      reject(error);
    } else {
      resolve(data);
    }
  });
});

promise.then((data) => {
  console.log(data);
}, (error) => {
  console.log(error);
});
```

## Promise的成员方法
### then()
前面的实例已经使用到了then方法，其接受两个参数：
> promise.then(resolve[, reject]);
> 
> @param Function resolve: 成功回调函数
> @param Function reject: 失败回调函数]
> @return Promise 新的Promise实例，并不是原来的Promise实例

then方法的作用是向Promise实例中添加状态改变时的回调函数。  
    
因为then方法返回一个新的Promise实例，所以支持链式调用。

### catch()
> catch(callback)
> 
> @param Function callback: 异步操作中发生错误时的回调函数
> @return Promise

catch(callback)的实际就是：

```js
then(null, callback);
```
其用于指定发生错误时的回调函数。当Promise的状态变为resolved时，调用then方法传入的成功回调函数，当异步操作发生错误时，Promise的状态变为rejected时，调用catch传入的回调函数。另外，then方法指定的回调函数，如果在运行中抛出错误，也会被catch方法捕获。

### finally()
finally方法中的代码不管最后Promise对象是什么状态都会执行。

> finally(callback)
> 
> @param Function callback: 回调函数
> @return Promise

其底层代码为：

```js
Promise.prototype.finally = function (callback) {
  let P = this.constructor;
  return this.then(
    value  => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => { throw reason })
  );
};
```
可见，finally也是通过then方法实现的。

### all()
> all(promiseArray)
> 
> @param Array promiseArray: Promise对象数组
> @return Promise

只有当Promise数组中的所有实例的状态都变为resolved时，或者其中一个变为rejected时，才会执行回调函数。例如：

```js
var firstPromise = new Promise((resolve, reject) => {
    var res1 = 1;
    console.log("res1:", res1);
    return Promise.resolve(res1);
});

var secondPromise = new Promise((resolve, reject) => {
    var res2 = 2;
    console.log("res2:", res2);
    return Promise.resolve(res2);
});

Promise.all([firstPromise, secondPromise])
    .then(([res1, res2]) => {
        console.log(res1);
        console.log(res2);
    });
```
上述代码中，当firstPromise和secondPromise的状态全部都为resolved时，执行后面的成功回调，当任意一个对象的状态为rejected时，执行后面的失败回调。

### race()
与all方法一样，race方法接受一个Promise数组作为参数，与all不同的是，当race的参数数组中任意一个Promise对象的状态变为resolved时，后续回调执行。

### Promise.resolve()
Promise.resolve()可以将现有对象转化为Promise对象。

```js
Promise.resolve(params);
// 等价于
new Promise(params => resolve(params));
```
#### 1、参数为thenable对象
thenable对象是指具有then方法的对象，Promise.resolve()会将该对象转化为Promise对象，然后立即执行该对象中的then方法，对象的状态就变为resolved。

#### 2、参数为Promise实例
如果参数是一个Promise实例，Promise.resolve()将直接返回这个实例。

#### 3、参数为其他值
参数作为成功回调函数的参数，返回一个状态为resolved的Promise对象

#### 4、无参数
只返回一个resolved状态的Promise对象

### Promise.reject()
返回一个rejected状态的Promise对象，参数作为后续方法的参数。

*Mission Complete!*