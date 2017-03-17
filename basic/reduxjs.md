# ReduxJS
认识`Redux`之前要理解React中的状态(`state`)。
```jsx
class App extends Component {
    state = {
        loading: false
    }
    render() {
        return (
            <Button loading={this.state.loading} onClick={() => {
                this.setState({loading: !loading});
            }}/>
        )
    }
}
```
在上面的例子中，`Button`的`loading`属性就是由`App`的状态决定的，点击`Button`可以改变状态从而决定`Button`是否处于`loading`中。


`ReduxJS`(简称`Redux`)可以创建一个在React应用管理状态(`state`)的容器，它有`Store`，`Action`，`Reducer`这些概念。而`Store`就是状态(`state`)的容器。

<a id="1"/>

## Action
`Action`是一个抽象的概念，可以理解为一个动作，比如按钮的点击，列表的选择。一个事件触发一个`Action`，以此来更改状态(`state`)。`Action`概念是以对象(`object`)的形式具现化的，而且有固定的格式：
```js
{ type: 'save', payload: { loading: true } }
```
> #### default::备注
>
>`type`是这个`Action`的类型，比如`save`就是保存状态`Action`；`payload`是‘负载’，即这个`Action`携带了什么，既然要保存，那肯定要有被保存的数据，`payload`就是携带这些数据的


而一个`Action`通常是通过`dispatch`方法来分发的，`dispatch`方法会自动映射到类的属性中。
```js
class App extends Component {
    render() {
        return (
            <Button loading={this.props.loading} onClick={() => {
                this.props.dispatch({type: 'save', payload: { loading: !this.props.loading }})
            }}/>
        )
    }
}
```

<a id="2"/>

## Reducer

`Reducer`是用来处理被`dispatch`的`Action`的。它是一个函数，接受`state`和`action`，返回一个新的`state`。
```js
function reducer(state, action) {
    if(action.type === 'save') {
        return { ...state, ...action.payload }
    }    
}
```

> #### default::备注
>
>具体store,action,reducer之间是如何交互的，查看[dva](../framework/dva.md)