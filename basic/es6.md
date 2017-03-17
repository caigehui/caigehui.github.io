# ES6

<a id="1"/>

## 变量声明 

**const 和 let**

不要用 `var`，而是用 `const` 和 `let`，分别表示常量和变量。不同于 `var` 的函数作用域，const 和 `let` 都是块级作用域。
```js
const DELAY = 1000;//常量

let count = 0;//块级变量
count = count + 1;
```

**模板字符串**

模板字符串提供了另一种做字符串组合的方法。
```js
const user = 'world';
console.log(`hello ${user}`);  // hello world

// 多行
const content = `
  Hello ${firstName},
  Thanks for ordering ${qty} tickets to ${event}.
`;
```

**默认参数**

在ES6中可以设置参数的缺省值
```js
function logActivity(activity = 'skiing') {
  console.log(activity);
}

logActivity();  // skiing
```
<a id="2"/>

## 箭头函数 
函数的快捷写法，不需要通过 `function` 关键字创建函数，并且还可以省略 `return`关键字。

同时，箭头函数还会继承当前上下文的 this 关键字，不需要进行绑定(`bind`)操作。

```js
class A extends React.Component {

    onClick = () => {
        console.log('clicked')
    }

    render() {
        return <div onClick={this.onClick}/>
    }
}

```
`function`写法：
```js
class A extends React.Component {

    onClick() {
        console.log('clicked')
    }

    render() {
        return <div onClick={this.onClick.bind(this)}>
    }
}
```
箭头函数还可以省略`return`关键字，当箭头右侧不指定花括号`{}`的时候，表示`return`箭头右测的表达式
```js
let foo = () => () => console.log('gg')
//等价于
let foo = () => {
    return () => {
        return console.log('gg');
    }
}
//等价于
function foo() {
    return function() {
        return console.log('gg');
    }
}

```
<a id="3"/>

## 模块的导入和导出
`import` 用于引入模块，`export` 用于导出模块。
例如:
```js
// 默认导出类
export default class App extends React.Component {};
// 或者
export default App;
// 引入默认
import App from 'app';

//导出部分
const HEIGHT = 100;
const WIDTH = 100;
export {
    HEIGHT,
    WIDTH
}
// 引入部分
import { HEIGHT, WIDTH } from 'constants';
// 引入全部并作为 CONSTANTS 对象
import * as CONSTANTS from 'constants';
//引用宽高
CONSTANTS.HEIGHT
CONSTANTS.WIDTH


```


<a id="4"/>

## ES6 对象和数组

**析构赋值**

析构赋值让我们从 Object 或 Array 里取部分数据存为变量。

```js
// 对象
const user = { name: 'guanguan', age: 2 };
const { name, age } = user;
console.log(`${name} : ${age}`);  // guanguan : 2

// 数组
const arr = [1, 2];
const [foo, bar] = arr;
console.log(foo);  // 1
```
我们也可以析构传入的函数参数。
```js
const add = (state, { payload }) => {
  return state.concat(payload);
};
```
析构时还可以配 alias，让代码更具有语义。
```js
const add = (state, { payload: todo }) => {
  return state.concat(todo);
};
```
**对象字面量改进**

这是析构的反向操作，用于重新组织一个 Object 。
```js
const name = 'duoduo';
const age = 8;

const user = { name, age };  // { name: 'duoduo', age: 8 }
```
定义对象方法时，还可以省去 function 关键字。
```js
app.model({
  reducers: {
    add() {}  // 等同于 add: function() {}
  },
  effects: {
    *addRemote() {}  // 等同于 addRemote: function*() {}
  },
});
```
**展开符号Spread Operator**

Spread Operator 即 3 个点 `...`，有几种不同的使用方法。

可用于组装数组。
```js
const todos = ['Learn dva'];
[...todos, 'Learn antd'];  // ['Learn dva', 'Learn antd']
```

也可用于获取数组的部分项。
```js
const arr = ['a', 'b', 'c'];
const [first, ...rest] = arr;
rest;  // ['b', 'c']

// With ignore
const [first, , ...rest] = arr;
rest;  // ['c']
```

还可收集函数参数为数组。
```js
function directions(first, ...rest) {
  console.log(rest);
}
directions('a', 'b', 'c');  // ['b', 'c'];
```

代替 apply。
```js
function foo(x, y, z) {}
const args = [1,2,3];

// 下面两句效果相同
foo.apply(null, args);
foo(...args);
```

对于 Object 而言，用于组合成新的 Object 。(ES2017 stage-2 proposal)
```js
const foo = {
  a: 1,
  b: 2,
};
const bar = {
  b: 3,
  c: 2,
};
const d = 4;

const ret = { ...foo, ...bar, d };  // { a:1, b:3, c:2, d:4 }
```

此外，在 JSX 中 Spread Operator 还可用于扩展 props，详见[展开属性](./reactjs.md#2)

<a id="5"/>

## Promise
Promise 用于更优雅地处理异步请求。比如发起异步请求：
```js
fetch('/api/todos')
  .then(res => res.json())
  .then(data => ({ data }))
  .catch(err => ({ err }));
```
定义 Promise 。
```js
const delay = (timeout) => {
  return new Promise(resolve => {
    setTimeout(resolve, timeout);
  });
};

delay(1000).then(_ => {
  console.log('executed');
});
```

<a id="6"/>

## Generators
[dva](../framework/dva.md) 的 effects 是通过 generator 组织的。Generator 返回的是迭代器，通过 yield 关键字实现暂停功能。`只有当yield后面的方法返回结果时才会执行下一行代码`

这是一个典型的 dva effect，通过 yield 把异步逻辑通过同步的方式组织起来。

```js
app.model({
  namespace: 'todos',
  effects: {
    *addRemote({ payload: todo }, { put, call }) {
      yield call(addTodo, todo);
      yield put({ type: 'add', payload: todo });
    },
  },
});
```