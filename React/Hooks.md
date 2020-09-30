# React Hooks

React v16.4 对 React 的架构进行了比较大的更改，具体可以参照[React Fiber](https://xiaobaihaha0001.gitbook.io/fe-share/react/react-sheng-ming-zhou-qi#react-tong-bu-geng-xin-de-ju-xian)

React 的新版本要求开发者不在第一阶段中的函数产生副作用。一个无副作用的函数只要接收的参数不变，那么最终返回的结果也是一样的。像 Ajax/Redux 等操作往往会产生副作用。所以 React 的新版本将旧版本在第一阶段的函数几乎完全废弃，只保留一个`shouldComponentUpdate`。

React 引入 Hooks 的原因一个是类的概念相对比较难懂，React 实际使用中也并没有使用面向对象的更多特点；另一方面，React 生命周期相对比较难以理解。

Hooks 是一个函数组件，在 Hooks 出来之前，React 也是支持函数型组件，例：

```jsx
const Header = ({ title, subtitle }) => {
  return (
    <header>
      <h1>{title}</h1>
      <h2>{subtitle}</h2>
    </header>
  );
};

// use Header

<div>
  <Header title="标题" subTitle="副标题">
</div>
```

可以看到，React 的函数型组件只能接收它的 Props 来渲染，相对类组件来说没法使用 State 来改变内部状态，也没有生命周期来对组件进行控制。

React Hooks 提供了一系列的 Hook 可以来让函数型组件功能更加丰富，在本文例子中，只使用 `useState`,`useEffect`,`useContext`, `useRef` 进行介绍。

## useState

`useState`可以来替换类组件中`this.state`的作用，例：

```jsx
import React, { useState } from "react";

const Header = ({ title, subTitle }) => {
  const [count, setCount] = useState(0);

  return (
    <header>
      <h1>{title}</h1>
      <h2>{subtitle}</h2>
      <span>{count}</span>
      <button onClick={() => setCount(count + 1)}>+</button>
    </header>
  );
};
```

`useState`接收一个初始值，返回一个数组，数组的第一个值为目前值的引用，第二个参数为进行值改变的回调，类似类组件中的`this.setState`。

## useEffect

`useEffect`可以用来替换 React 的生命周期，React 中的生命周期使用较多的为 componentDidMount,componentDidUpdate。

`useEffect` 接收两个参数，第一个参数为一个函数，第二个参数为一个依赖的数组，当第二个参数中的数组的值发生改变时，会执行第一个参数的函数。 `useEffect` 默认第一次渲染也会执行。

- componentDidMount / componentDidUpdate 都需要执行的逻辑

`useEffect`也可以不接受第二个参数，它会在每次重新渲染中都会执行一遍，也就是无论是 Props 或者是内部状态的改变，都会进行重新执行。所以在第二个参数为空的情况下，可以模拟 componentDidMount 和 componentDidUpdate 都需要执行的一些逻辑。

```jsx
useEffect(() => {
  // ...
});
```

- componentDidMount

由于`useEffect` 默认第一次渲染也会执行，所以只需要第二个参数为空数组即可模拟 componentDidMount。

```jsx
useEffect(() => {
  // ...
}, []);
```

- componentDidUpdate

`useRef`可以创建一个 ref，这个往往是用作来创建 DOM 的 Ref，但是在这里也可以借助它来作为一个函数的成员变量来标示是否是 Update 阶段。

```jsx
const mounted = useRef();

useEffect(() => {
  if (!mounted.current) {
    mounted.current = true;
  } else {
    // ...
  }
});
```

## useContext

React 类组件中使用 Context 需要使用 `XXXContext.Consumber` 进行使用，具体可以参照[React 数据流](https://xiaobaihaha0001.gitbook.io/fe-share/react/react-shu-ju-liu)。在 Hooks 中，可以更加方便地使用 Context。

```jsx
const Context = React.CreateContext();

// Class
render() {
  <Context.Consumer>
  {value => {
    return <div>{value}</div>
  }}</Context.Consumer>;
}

// Hooks
const contextValue = useContext(Context);
return <div>{contextValue}</div>;
```
