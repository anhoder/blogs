# 简单介绍pjax

<!-- more -->

## 一、pjax是什么
pjax是什么? 它是pushState和ajax的合称，相当于是ajax的plus版。
## 二、pjax是干什么的
说到pjax的作用，就不得不和ajax作比较的。
### 1. ajax的缺点
> ajax缺点：对于每一次的ajax请求，浏览器都不会将其保存在历史记录中，因此当用户返回或前进时，将直接跳出该页面。

举个🌰：
打开浏览器，在地址栏中输入URL，回车：
![1.jpg](https://media.alan123.xyz/imgs/blogs/pjax/1.jpg)
然后点击按钮进行ajax请求：
![2.jpg](https://media.alan123.xyz/imgs/blogs/pjax/2.jpg)
可以发现请求前后的URL是保持不变的，如果这时点击浏览器的返回键，页面就会回到了最初的状态。
   
试想一下，如果我们构建一个web应用使用到ajax请求，那就可能意味着我们强行将用户浏览器的返回和前进按给钮扣掉了。
### 2. pjax的解决原理
pjax就是为了解决ajax上述问题、提高用户的体验而出现的，其使用的关键技术就是HTML5的pushState。

#### pushState又是什么呢？
pushState是HTML5中对history对象新增的方法，所以我们可以得知它是用来操作历史记录的方法，主要作用就是在不刷新网页的情况下改变URL。

使用语法：

```js
pushState(state, title, url);
```

又一个🌰：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>测试</title>
    <script src="https://media.alan123.xyz/jquery/3.3.1/jquery.min.js"></script>
</head>
<body>
    <button id="magic">变个魔术</button>
    <script type="text/javascript">
        $('#magic').click(function(event) {
            history.pushState(null, null, Math.random() * 100);
        });
    </script>
</body>
</html>
```
点击“变个魔术”按钮，你会发现页面并没有刷新，但URL却已经改变了，这时你可以点击返回，URL会变成上一个URL，效果图如下：
![3.jpg](https://media.alan123.xyz/imgs/blogs/pjax/3.jpg)

#### pjax的使用
OK，了解了pushState，我们大致清楚了pjax的实现原理，接下来开始使用pjax，因为已经有人造好了轮子，所以我们可以直接使用，给个[传送门](https://github.com/defunkt/jquery-pjax)。
使用方法：

```html
$(document).pjax(element1, element2);
// element1为触发事件的元素，element2为装载返回内容的容器
```

最后一个栗子：
① test.html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>测试</title>
    <script src="http://media.alan123.xyz/jquery/3.3.1/jquery.min.js"></script>
    <script src="http://media.alan123.xyz/jquery/pjax/jquery.pjax.js"></script>
</head>
<body>
    <h3 id="content">这是标题！</h3>
    <a href="test.php">ajax请求</a>
    <script type="text/javascript">
        $(document).pjax('a', '#content');
    </script>
</body>
</html>
```
② test.php

```php
<?php
echo 'Hello World';
```
结果就不上图，自己测试吧


