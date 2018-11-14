# React Native篇---（三）state

## 状态机state

### 写法一

状态机state是React框架中的一个重要概念，**每种UI场景对应状态机的一种状态**。随着状态机变量的改变，UI场景也在重新渲染。例如：

```jsx
export default class MyProject extends Component {
  constructor (props) {
    super(prpos);
    // 构造方法中，声明需要用到的状态机变量
    this.state = {
      textContent: ''
    };
  }
  
  // 更新状态机方法
  updateTextContent (newText) {
    // 使用setState函数更新状态机
    this.setState((state) => {
      return {
        textContent: newText
      };
    });
  }
  
  // 渲染
  render () {
    return (
      <View>
        {/* onChangeText触发时调用箭头函数 */}
        <TextInput 
          onChangeText={(newText) => this.updateTextContent(newText)} />
        {/* 使用this.state.变量名的形式获取状态机的变量值 */}
        <Text>输入框内容：{this.state.textContent}</Text>
      </View>
    )
  }
}
```
上述代码中，给TextInput组件的onChangeText绑定一个箭头回调函数（onChangeText的回调函数接受改变后的文本作为参数），箭头函数内部调用updateTextContent方法。

### 写法二

```jsx
export default class MyProject extends Component {
  constructor (props) {
    ...
    // 前面不变，后面为增加代码
    this.updateTextContent = this.updateTextContent.bind(this);
  }
  
  // 保持不变
  updateTextContent (newText) {
    ...
  }
  
  render () {
    return (
      <View>
        {/* 改变部分 */}
        <TextInput onChangeText={this.updateTextContent} />
        ...
      </View>
    );
  }
}
```
也就是说，如果你想要使用this.updateTextContent作为onChangeText的回调函数，你需要在构造方法中结合this.updateTextContent = this.updateTextContent.bind(this)使用。那这条语句到底什么作用呢？我们来测试一下：

> ① 如果不加上该语句，当输入框的内容变化时，会报错：this.setState is not a function。   
>
> ② 我们在不加上面那条语句的基础上，在updateTextContent中使用console.log输出this。打开Remote JS Debugging，改变输入框内容，输出为：   
> 
> ![结果1](https://media.alan123.xyz/imgs/blogs/react-native/3/1.png)
> 
> 可见，this包含allowFontScaling和onChangeText，可以推测出这个“this”代表TextInput组件。而TextInput组件是没有setState方法的，所以才会出现上面的报错。    

> ③ 加上那条语句，再次输出this，结果为：

> ![结果2](https://media.alan123.xyz/imgs/blogs/react-native/3/2.png)

> 在这，“this”代表RNApp。   

> 由此可知绑定操作是为了让回调函数能够正确的解析出this。

### 写法三
updateTextContent函数若使用箭头函数的形式定义，则不需要进行绑定操作。如：

```jsx
updateTextContent = (newText) => {
  this.setState({
    textContent: newText
  });
}

render () {
  return (
    <View>
      <TextInput onChangeText={this.updateTextContent} />
    </View>
  );
}
```
这是因为JavaScript中每个普通函数都有自己的this值，而箭头函数会捕获其上下文的this值作为其自身的this值。*详细内容见[这篇文章]()*

## setState函数

```jsx
setState(partialState, callback); // callback为成功设置State后的回调函数，可选
```

例如：

```jsx
updateTextContent (newText) {
  this.setState({
    textContent: newText
  }, () => {
    console.log("textContent has changed");
  });
}
```

也可以降低一个参数换为函数，其接受oldState作为参数，且需返回改变后的状态机变量，如：

```jsx
updateTextContent (newText) {
  this.setState((oldState) => {
    console.log(oldState); // 输出为：textContent: ""
    return {
      textContent: newText
    };
  }, () => {
    console.log("textContent have changed.");
  });
}
```


## 注意事项

> ① 勿将上述代码中的回调函数写成如下形式：
> 
> ```jsx
> onChangeText = {this.updateTextContent(newText)}
> ```
> this.updateTextContent代表函数，其值就是整个函数；而this.updateTextContent()表示执行函数，其值为执行函数后的返回值
> 
> ② 更新状态机变量时，勿使用this.state.var = value;形式，而应该使用this.setState方法进行更新。虽然从语法上来说this.state的方式是正确的，但从React原则是不合法的。读取状态机变量可直接使用this.state的形式。
> 
> ③ 尽量让自定义的组件不使用状态机变量，设计思路是：将多个无状态的组件封装在一个有状态的组件中，并把有状态的组件的状态机变量通过props传给无状态的组件。有状态的组件负责UI的交互逻辑，无状态的组件负责UI的渲染。
> 
> ④ 尽可能让UI中可变的数据来源是状态机变量与属性。
> 
> ⑤ 在状态机变量中存放与显示无关的变量，会导致React Native无必要地判断是否需要重新渲染UI，从而导致应用性能下降。


*Mission Complete!*