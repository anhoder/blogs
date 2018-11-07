# React篇---（一）简介



## Virtual DOM

在传统的前端开发中，我们在更新页面时是使用JavaScript直接操作DOM，而DOM操作的性能消耗是前端开发中最大的，并且代码也会变得难以维护。



React引进了Virtual DOM，其将真实的DOM转化为JavaScript对象树，这将会带来以下优势：

>  ① 性能提升：每次数据更新后，重新计算Virtual DOM，并和上次的Virtual DOM做对比，对发生变化的部分做批量更新。
>
>  ② 方便与其他平台集成：例如React Native是基于Virtual DOM渲染出的原生控件，因为React组件可以映射为对应的原生控件。在输出时，具体是Web DOM、Android控件或iOS控件，由平台本身决定。

对于Virtual DOM，举个例子：

```html
<div>
  <button class="btn">提交<button>
</div>
```

上述HTML DOM可以转换为如下的JavaScript对象树：

```javascript
{
  type: 'div',
  props: {
    children: {
      type: 'button',
      props: {
        className: 'btn',
        children: '提交'
      }
    }
  }
}
```

注意：

> 因为属性class、for在JavaScript中是保留字，因此，分别使用className、htmlFor进行替代。



## JSX

使用JavaScript对象树去描述真实的DOM是很贴切的，可是在上面的例子中，使用HTML只是一小段，转化为JavaScript对象树却是很冗长的代码。如果要描述更复杂的DOM呢？另外一方面，JavaScript对象树的可读性较差，没有HTML来的简单直接。JSX正是在这样的环境下应运而生。例如：

```jsx
const RegisterForm = () => (
  <form method="post" action="/reg" >
    <input type="text" name="userName" />
    <input type="password" name="userPwd" />
    <input type="submit" value="提交" />
  </form>
);
```

上述代码中，JSX将HTML代码直接加入到JavaScript中，通过翻译器转化为纯JavaScript再由浏览器执行。JSX类似于CoffeeScript，会在产品上线之前，打包编译为纯JavaScript代码。



### XML
JSX是使用的是类XML语法，在使用时需要注意的有：
> ① 定义标签时，只允许被一个标签包裹，如：```const Example = <div></div><div></div>```会报错。
> ② 标签一定要闭合，否则会报错，这和XML的语法一致。



### DOM元素和组件元素

DOM元素是指HTML中的标签，例如：```<div></div>```、```<span></span>```。

组件元素是根据具体的功能将DOM元素组合在一起形成的元素，例如：```const Example = <div><button>例子</button></div>```，Example就是一个组件元素，其可以在其他地方被使用：```<Example></Example>```。

DOM元素的命名规范：

> DOM元素的标签首字母为小写，组件元素的标签首字母为大写。



### 使用变量

在JSX中使用变量需要用{}将变量包裹住，例如：

```jsx
const Example = (param) => (
  <div>{param}</div>
);
```



### 注释

在HTML中，注释使用```<!-- 注释 -->```的形式，而在JSX中，使用```{/* 注释 */}```作为注释。例如：

```jsx
const Example = (<div>{/* 这是注释 */}</div>);
```

可以理解为，{}中为JavaScript代码，使用的是JavaScript注释方式。



### 自定义属性

在JSX中，往DOM元素中传入自定义属性会被忽略，不被渲染：

```jsx
<div cus="example"></div>
```

如果需要使用自定义属性，可以是加上data-前缀（与HTML一致）：

```jsx
<div data-cus="example"></div>
```

但在自定义的标签中，任何属性都会被支持。



### 转义字符

React会将显示到DOM的字符串进行转义，防止XSS。所以，如果在JSX中加入```&copy;```并不会显示为&copy;，如果需要使用特殊字符，可以使用一下方法：

> ① 直接使用UTF-8的字符
>
> ② 使用对应字符的Unicode编码
>
> ③ 使用数组组装```<div>{['cc ', <span>&copy;</span>, ' 2018']}</div>```
>
> ④ 直接插入原始 的HTML

