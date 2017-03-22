# ReactJS

<a id="1"/>

## Stateless Functional Components 
React Component 有 3 种定义方式，分别是 `React.createClass`, `class` 和 `Stateless Functional Component`。推荐尽量使用最后一种，保持简洁和无状态。这是函数，不是 `Object`，没有 `this` 作用域，是 `纯函数`。

比如定义 App Component 

```js
function App(props) {
  function handleClick() {
    props.dispatch({ type: 'app/create' });
  }
  return <div onClick={handleClick}>${props.name}</div>
}
```
等同于：
```js
class App extends React.Componnet {
  handleClick() {
    this.props.dispatch({ type: 'app/create' });
  }
  render() {
    return <div onClick={this.handleClick.bind(this)}>${this.props.name}</div>
  }
}
```

<a id="2"/>

## JSX语法规则

**Component 嵌套**

类似 HTML，JSX 里可以给组件添加子组件

```js
<App>
  <Header />
  <MainContent />
  <Footer />
</App>
```

**className**

`class` 是保留词，所以添加样式时，需用 `className` 代替 `class` 。
```jsx
<h1 className="fancy">Hello dva</h1>
```

**JavaScript 表达式**

JavaScript 表达式需要用 {} 括起来，会执行并返回结果。

比如：
```js
<h1>{ this.props.title }</h1>
```
**Mapping Arrays to JSX**

可以把数组映射为 JSX 元素列表。

```js
<ul>
  { this.props.todos.map((todo, i) => <li key={i}>{todo}</li>) }
</ul>
```
**注释**

尽量别用 // 做单行注释。

```jsx
<h1>
  {/* multiline comment */}
  {/*
    multi
    line
    comment
    */}
  {
    // single line
  }
  Hello
</h1>
```

**展开属性Spread Attributes**

这是 JSX 从 ECMAScript6 借鉴过来的很有用的特性，用于扩充组件 props 。

比如：

```jsx
const attrs = {
  href: 'http://example.org',
  target: '_blank',
};
<a {...attrs}>Hello</a>
```
等同于
```js
const attrs = {
  href: 'http://example.org',
  target: '_blank',
};
<a href={attrs.href} target={attrs.target}>Hello</a>
```
 
<a id="3"/>

## Props属性
数据处理在 React 中是非常重要的概念之一，分别可以通过 props, state 和 context 来处理数据。而在 [dva](../framework/dva.md) 应用里，你只需关心 props 。

**propTypes和defaultProps**

JavaScript 是弱类型语言，所以请尽量声明 propTypes 对 props 进行校验，以减少不必要的问题。

```jsx
function App(props) {
  return <div>{props.name}</div>;
}
App.propTypes = {
  name: React.PropTypes.string.isRequired,
};
App.defaultProps = {
    name: 'abc'
}
```

内置的 prop type 有：

* PropTypes.array
* PropTypes.bool
* PropTypes.func
* PropTypes.number
* PropTypes.object
* PropTypes.string

<a id="4"/>

## CSS Modules

在React中使用CSS是通过CSS Modules来实现的，例如：

app.css:
```css
. button {
    border-radius: 4px;
    background-color: LightCyan;
}
```

app.js
```jsx
import React from 'react';
import styles from './app.css';

function App(props) {
    return (
        <div className={styles.button}>button</div>
    )
}
export default App;
```

**Inline Styles**

还有一种定义样式的方法，使用内联样式( inline style)，定义styles的时候使用的对象的形式，样式名称跟css略有不同，移除了`-`，使用驼峰法命名，例如：
```js
import React from 'react';

const styles = {
    button: {
        backgroundColor: '#fff',
        borderRadius: 4
    }
}
function App(props) {
    return (
        <div style={styles.button}>button</div>
    )
}
export default App;
```

**CSS终极解决方案**

引入[reactCSS](http://reactcss.com/)库

定义样式
```jsx
import reactCSS from 'reactcss'

const styles = reactCSS({
  'default': {
    card: {
      background: this.props.background,
      boxShadow: '0 2px 4px rgba(0,0,0,.15)',
    },
  },
})
```
内联方式引用
```jsx
<div style={ styles.card } />
```