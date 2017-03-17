# react-wxeap-mobile

react-wxeap-mobile是WxEAP平台的前端框架

它基于[dva](https://github.com/dvajs/dva)集成了`redux`, `redux-saga`, `react-router`，
将状态和路由进行统一管理，使用非常简单

## 安装
> npm i react-wxeap-mobile --save

<a id="1">

## 配置路由和数据模型

在入口文件进行配置
```js
import { App } from 'react-wxeap-mobile';
import './index.css';

App([
  {
    path: '/',
    model: require('./models/tasksList'),
    component: require('./routes/TasksList')
  }
])({
  module: 'wxcsm',
  origin: 'http://192.168.0.92/WxSoft.EAP',
  auth: '/WxLoginIF.aspx?EmpNo=sy&EmpPassword=111111'
})
```

> App(routes)(config)
routes是所有路由配置的数组，config是配置api的对象

在代码中使用`dispatch`或者`put`进行路由跳转
```js
    dispatch(routerRedux.push({
      pathname: '/TaskDetail',
      query: {
        taskId,
        taskTitle
      }
    }))
    yield put(routerRedux.push({
      pathname: '/TaskDetail',
      query: {
        taskId,
        taskTitle
      }
    })
```

> #### default::query
>
> query是查询对象，不支持对象嵌套，在浏览器中的形式为 http://.../#TaskDetial?taskId=1&taskTitle=xxx


<a id="2"/>

## service 

`service`是存放网络请求的，一个业务对应一个方法，返回网络请求的结果
```js
import { request } from 'react-wxeap-mobile';

export function query() {
  return request('/api/users');
}

```
在model的effects中使用，当出错时，data为null，当正常时，err为null
```js
const { data, err } = yield call(query);
```
 
## model
`model`是用来存放一个视图的所有业务逻辑
```js
export default {
  namespace: 'taskList',
  state: {
      data: {},
      page: 1
  },
  reducers: {
    save(state, { payload }) {
      return { ...state, ...payload }
    },
  },
  effects: {
    *initFetch({ payload: { areId } }, { call, put, select }) {
        const { page } = yield select(state => state.taskList);
        const { data } = yield call(fetchData, { page, areId });
        yield put({ type: 'save', payload: { data } })
    }
  },
  subscriptions: {
    setup({ dispatch, history }) {
      return history.listen(({ pathname, query }) => {
        if (pathname === '/') {
          dispatch({ type: 'initFetch', payload: query });
        }
      });
    },
  },
};

```
上面的代码是某个任务列表视图的模型，一旦路由到这个视图，那么他将自动获取数据，自动绑定到视图

model包含下面五个属性：

**namespace**

模型名称，在状态容器(Store)的对象名，用于索引状态树

**state**

`state`定义了一个视图的初始状态，例如data: {}，那么这个视图的初始数据就为空

**reducers**

`reducers`是reducer的集合，用于处理action，dva会自动根据`action`的`type`映射到reducers的某个函数中

以 key/value 格式定义 reducer。用于处理同步操作，唯一可以修改 state 的地方。由 action 触发。

格式为 `(state, action) => newState`
```js
dispatch({ type: 'save', payload: {  } })
//映射到reducers的save方法
save(state, action) {

}
```
**effects**

`effects`是副作用，它的概念涉及到函数编程思想，这里不理解也没关系，把它当成reducers的一种，是用来进行异步网络请求的

以 key/value 格式定义 effect。用于处理异步操作和业务逻辑，不直接修改 state。由 action 触发，可以触发 action，可以和服务器交互，可以获取全局 state 的数据等等。

格式为`*(action, effects) => void`
```js
dispatch({ type: 'initFetch', payload: { areId: 1 } })

//映射到effects的initFetch
*initFetch({ payload: { areId } }, { select, call, put }) {
    const { page } = yield select(state => state.taskList);
    const { data } = yield call(fetchData, { page, areId });
    yield put({ type: 'save', payload: { data } })
}
```

这里`select`, `call`, `put`是`dva`提供的异步工具

* `select`用于获取当前的`state`
* `call`用于调用需要时间返回结果的方法，第一个参数是方法，后面的参数是该方法的参数
* `put`的作用和`dispatch`相同

**subscriptions**

以 key/value 格式定义 subscription。subscription 是订阅，用于订阅一个数据源，然后根据需要 dispatch 相应的 action。在 `app.start()` 时被执行，数据源可以是当前的时间、服务器的 websocket 连接、keyboard 输入、geolocation 变化、history 路由变化等等。

格式为 `({ dispatch, history }, done) => unlistenFunction。`

```js
setup({ dispatch, history }) {
      return history.listen(({ pathname, query }) => {
        if (pathname === '/') {
          dispatch({ type: 'initFetch', payload: query });
        }
      });
    },
```

<a id="3"/>

## route

route是我们看到的视图，它在入口文件作配置

**给route绑定model**

在ES6中，`React`使用高阶组件来代替`mixins`，两者都可以给一个对象赋予额外的属性，与视图绑定便利用了这个特性

```js
import { connect } from 'react-wxeap-mobile';

function TaskDetail({ text }) {
    return <div>{text}</div>
}

const mapStateToProps = state => {
    return {
        text: state.text
    }
}

export default connect(mapStateToProps)(TaskDetail);
```

`connect`就是给`TaskDetail`这个视图注入`model`定义的`state`

`mapStateToProps`用于映射`state`到`TaskDetail`的`props`中