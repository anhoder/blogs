# React Native篇---（四）组件的生命周期

*Mission Start!*

每个React Native组件从加载到卸载都经历了一个完整的生命周期。在这个生命周期中，React提供了一些钩子函数，开发者可以实现这些函数，来完成在某个特定时间执行一些操作。各周期函数如图：   

![React Native周期函数](https://media.alan123.xyz/imgs/blogs/react-native/4/1.svg)

<span style="color: red;">*Tips：上述的getDefaultProps和getInitialState分别是ES5使用的定义默认属性props的函数和定义初始状态state的函数。*</span>

## getDefaultProps

> getDefaultProps ()
> 
> return: 对象，设置的默认props

设置组件的一些默认属性。定义后可以使用this.props获取在这里初始化的属性。组件初始化后，再次使用该组件不会调用getDefaultProps，所以组件不能修改自己的props，只能由其他组件在调用它时在外部修改。

```jsx
getDefaultProps () {
  return {
    name: 'Alan'
  };
}
```

该函数只能在ES5中配合createClass使用，不能使用在ES6中。在ES6中定义默认属性：

```jsx
export default class MyProject extends Component {
  static defaultProps = {
    name: 'Alan'
  };
  // ...
}
// 或
export default class MyProject extends Component {
  // ...
}
MyProject.defaultProps = {
  name: 'Alan'
};
```

## getInitialState

> getInitialState ()
> 
> return: 对象，设置的初始state

该函数对组件的状态state进行初始化。该函数不同于getDefaultProps，其在后续过程中会被再次调用。

```jsx
getInitialState () {
  return {
    name: 'Alan'
  };
}
```

该函数也是只能在ES5中使用，在ES6中初始化state：

```jsx
export default class MyProject extends Component {
  constructor (props) {
    super(props);
    this.state = {
      name: 'Alan'
    };
  }
}
```

## constructor

> constructor (props)

React Native组件的构造函数，其第一条语句必须是super(props)。该周期函数在组件加载前最先调用，且只调用一次。可在该函数中定义状态机变量。如：

```jsx
constructor (props) {
  super(props);
  this.state = {
    text: ''
  };
}
// 以上为ES6定义初始state的方法
// ES5使用getInitialState进行定义
getInitialState () {
  return {
    text: ''
  };
}
```

## render

> render ()
> return: JSX组件 

render是一个组件中必须有的方法，返回JSX或其他组件构成DOM（只能返回一个顶级元素）。

## componentWillMount

> componentWillMount ()

该函数在组件渲染（render函数执行）前被调用执行，且在一个生命周期中只执行一次。子组件的componentWillMount函数在父组件的ComponentWillMount函数之后被调用。

## componentDidMount

> componentDidMount ()

该函数在组件渲染完成后被调用，且在一个生命周期中只执行一次。子组件的componentDidMount函数在父组件的componentDidComponent函数之前执行。

## componentWillReceiveProps

> componentWillReceiveProps (newProps)
> 
> param: 接受一个对象作为参数，该对象为新的props
> 
> *Tips：函数内，this.props表示旧的props，新的props为参数newProps*

在初次渲染完成后，当组件接收到新的props时，这个函数被调用。如果新的props会造成界面重新渲染，该函数会在渲染前执行。    
   
<span style="color: red">**在初次渲染时，该函数不会被执行**</span>

## shouldComponentUpdate

> shouldComponentUpdate (newProps, newState)
> 
> param: 接受两个对象作为参数，分别为新的props、新的state
> 
> return: 需返回Bool值，是否重新渲染该组件
> 
> *Tips：函数内，this.props(this.state)表示旧的props(state)，新的props(state)为参数newProps(newState)*

在初始渲染完成后，当组件接收到新的state或props时，该函数被调用。React Native默认返回true。

## componentWillUpdate

> componentWillUpdate (newProps, newState)

初次渲染完成后，在该组件重新渲染前被调用。

## componentDidUpdate

> componentDidUpdate (oldProps, oldState)
> 
> param：渲染前的props、state

初次渲染完成后，当该组件重新渲染完成会被调用。

## componentWillUnmount

> componentWillUnmount ()

在组件被卸载前，该函数被调用。

## 获取真实的DOM节点

在原生的JS中，获取DOM使用getElementById等一系列函数。而在React中，获取DOM前，需要给组件指定ref属性，通过ref属性获取DOM。如：

```jsx
constructor (props) {
  super (props);
  this.pressText = this.pressText.bind(this);
}

pressText () {
  console.log(this.refs.myTextInput);
}

render () {
  return (
    <View>
      <TextInput ref="myTextInput" />
      <Text onPress={this.pressText}>点击</Text>
    </View>
  );
}
```

## React中ES5、ES6的差别

### 创建类 

```jsx
// ES5(React 15.4及之前版本)
var React = require('react');
var Example = React.createClass({
  render () {
    return (<View></View>);
  }
});

// ES5(React 15.5之后的版本，React的createClass被移除)
var React = require('react');
var createReactClass = require('create-react-class');
var Example = createReactClass({
  render () {
    return (<View></View>);
  }
});

// ES6
import React, {Component} from 'React';
class Example extends Component {
  render () {
    return (<View></View>);
  }
}
```

### 设置默认props

```jsx
// ES5
getDefaultProps () {
  return {
    text: "This is default props"
  };
}

// ES6
static defaultProps = {
  text: "This is default props"
};
// 或
class MyProject extends Component {
  // ...
}
MyProject.defaultProps = {
  text: "This is default props"
};
```

### 设置初始state

```jsx
// ES5
getInitialState () {
  return {
    text: "This is initial state"
  };
}

// ES6
constructor (props) {
  super (props);
  this.state = {
    text: "This is initial state"
  };
}
```

*Mission Complete!*