# Vue.js的数据绑定原理

<!-- more -->

最近接触到Vue.js，首先碰到的就是数据绑定，对其原理实现十分好奇，于是深入了解一下。

*Mission Start!*

## 一、实现原理与技术
Vue.js的数据绑定是通过ES5中的Object.defineProperty() + 订阅者-发布者模式实现的。其通过object.defineProperty()劫持各个属性的set、get事件，在数据发生变动时发送消息给订阅者，触发相应的监听回调。   
   
在此也顺带捎一下简单介绍一下其他框架的实现：   
**Backbone.js**: 在Model到View的数据传递过程中，可以在View中监听Model的change事件，当Model数据更新时，刷新View；在View到Model的数据传递过程中，可以监听View的DOM元素的事件，当检测到View状态变更后，将变更的数据发送到Model。   
**Angular.js**: 采用"脏值检测"的方式，数据发生变更后，对于所有数据和视图的绑定关系进行一次检测，识别是否有数据发生改变，有改变进行处理，可能进一步引发其他数据的改变，进入循环，直至没有数据发生变化，然后将数据更新到视图。若手动对ViewModel的数据进行变更，则需要手动触发一次"脏值检测"。

### 1、Object.defineProperty()
defineProperty()是Object类的一个方法，其作用是在一个对象上定义一个新的属性或修改其原有属性，并返回这个对象。

#### 用法：

```js
Object.defineProperty(obj, prop, descriptor);
```
defineProperty方法接受三个参数和返回值：   
> ① **obj**: 目标对象   
> ② **prop**: 定义或修改的属性名称   
> ③ **descriptor**: 被定义或被修改的属性描述符（对象）   
> 返回值： 被传递给函数的经过修改的对象   
   
其中参数descriptor(描述符)主要有两种形式：**数据描述符**和**存取描述符**。   
**数据描述符**是一个具有值的属性，该值可能是可写的，也可能不是可写的。   
**存取描述符**是由getter-setter函数对描述的属性。描述符必须是两种形式之一，且不能同时为两者。

> **① 数据描述符和存取描述符都具有的键值**   
> **configurable**: 当且仅当该属性的configurable为true时，该属性描述符才能够被改变，同时该属性也能从对应的对象上被删除。**默认为false**。   
> **enumerable**: 当且仅当该属性的enumerable为true时，该属性才能出现在对象的枚举属性中（在for...in和Object.keys()）。**默认为false**。   
> **② 数据描述符具有的键值**   
> **value**: 该属性对应的值。可以是任何有效的JavaScript值。**默认为undefined**。    
> **writable**: 当且仅当该属性的writable为true时，value才能被改变。**默认为false**。   
> **③ 存取描述符具有的键值**   
> **get**: 一个给属性提供getter的方法，若没有getter则为undefined。当访问该属性时，该方法被执行，方法执行时没有参数传入，但是会传入this对象（由于继承关系，这里的this并不一定是定义该属性的对象）。**默认为undefined**。   
> **set**: 一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值。**默认为undefined**。   
> <span style="color:red;">*综上，数据描述符可含有的键值为：configurable、enumerable、value、writable；存取描述符可含有的键值为：configurable、enumerable、get、set；且数据描述符和存取描述符能且只能存在一种。*</span>

例如：   

```js
var obj = {};
Object.defineProperty(obj, 'x', {
    value: 123,
    writable: true,
    configurable: true,
    enumerable: true
});
/*
 * 上述代码为obj对象定义一个名为x的属性，并赋值为123（数据描述符）
 */

var obj2 = {
    x: 123
};
Object.defineProperty(obj2, 'x', {
    get: function () {
        return 'This is x';
    },
    set: function (value) {
        console.log("update x: " + value);
    }
});
/*
 * 以上代码为obj2创建属性x的存取描述符，当获取obj2的x属性时，返回"This is x"；当设置obj2的x属性时，控制台输出更新信息。
 */
```

### 2、订阅者-发布者模式
订阅者把自己想订阅的事件注册到调度中心，当该事件触发时候，发布者发布该事件到调度中心，由调用中心统一调度订阅者注册到调度中心的处理代码。如下图：
![订阅者-发布者模式](https://media.alan123.xyz/imgs/blogs/vue/1.png)

> 在此顺带介绍一下观察者模式：观察者把自己注册到目标（发布者）里，在具体目标发生变化时候，调度观察者的更新方法。如下图：
> ![观察者模式](https://media.alan123.xyz/imgs/blogs/vue/2.png)

<span style="color:red">*观察者模式与订阅者-发布者模式的区别：最大的区别是调度的地方。虽然两种模式都存在订阅者和发布者，但观察者模式是由目标调度的，而订阅者-发布者模式是由调度中心统一调度的，因此观察者模式的观察者和目标之间会存在着依赖，而订阅者-发布者不会，且订阅者-发布者是松耦合的。*</span>



