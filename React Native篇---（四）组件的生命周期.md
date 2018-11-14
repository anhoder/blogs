# React Native篇---（四）组件的生命周期

*Mission Start!*

每个React Native组件从加载到卸载都经历了一个完整的生命周期。在这个生命周期中，React提供了一些钩子函数，开发者可以实现这些函数，来完成在某个特定时间执行一些操作。

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
```

## componentWillMount

> componentWillMount ()

该函数在组件渲染（render函数执行）前被调用执行，且在一个生命周期中只执行一次。子组件的componentWillMount函数在父组件的ComponentWillMount函数之后被调用。

## componentDidMount

> componentDidMount ()

该函数在组件渲染完成后被调用，且在一个生命周期中只执行一次。子组件的componentDidMount函数在父组件的componentDidComponent函数之前执行。

## componentWillReceiveProps

> componentWillReceiveProps (newProps)
> param: 接受一个对象作为参数，该对象为新的props
> *Tips：函数内，this.props表示旧的props，新的props为参数newProps*

在初次渲染完成后，当组件接收到新的props时，这个函数被调用。如果新的props会造成界面重新渲染，该函数会在渲染前执行。    
   
<span style="color: red">**在初次渲染时，该函数不会被执行**</span>

## shouldComponentUpdate

> shouldComponentUpdate (newProps, newState)
> param: 接受两个对象作为参数，分别为新的props、新的state
> return: 需返回Bool值，是否重新渲染该组件
> *Tips：函数内，this.props(this.state)表示旧的props(state)，新的props(state)为参数newProps(newState)*

在初始渲染完成后，当组件接收到新的state或props时，该函数被调用。React Native默认返回true。

## componentWillUpdate

> componentWillUpdate (newProps, newState)

初次渲染完成后，在该组件重新渲染前被调用。

## componentDidUpdate

> componentDidUpdate (oldProps, oldState)
> param：渲染前的props、state

初次渲染完成后，当该组件重新渲染完成会被调用。

## componentWillUnmount

> componentWillUnmount ()

在组件被卸载前，该函数被调用。

*Mission Complete!*