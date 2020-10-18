# React Ref

在 React 数据流中，props 是父组件与子组件交互的唯一方式。要修改一个子组件，你需要使用新的 props 来重新渲染它。但是，在某些情况下，你需要在典型数据流之外强制修改子组件/元素。

适合使用 refs 的情况：

- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。

## 字符串获取：

```jsx
// string ref
class MyComponent extends React.Component {
  componentDidMount() {
    this.refs.myRef.focus();
  }

  render() {
    return <input ref="myRef" />;
  }
}
```

## 回调函数

```jsx
// callback ref
class MyComponent extends React.Component {
  componentDidMount() {
    this.myRef.focus();
  }

  render() {
    return (
      <input
        ref={(ele) => {
          this.myRef = ele;
        }}
      />
    );
  }
}
```

## React.createRef

```jsx
// React.createRef
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }

  componentDidMount() {
    this.myRef.current.focus();
  }

  render() {
    return <input ref={this.myRef} />;
  }
}
```

### React.createRef 的优点

相对于 callback ref 而言 React.createRef 显得更加直观，避免了 callback ref 的一些理解问题。

### React.createRef 的缺点

性能略低于 callback ref; ref 的值根据节点的类型而有所不同：

当 ref 属性用于 HTML 元素时，构造函数中使用 React.createRef() 创建的 ref 接收底层 DOM 元素作为其 current 属性。
当 ref 属性用于自定义 class 组件时，ref 对象接收组件的挂载实例作为其 current 属性。
默认情况下，你不能在函数组件上使用 ref 属性（可以在函数组件内部使用），因为它们没有实例。如果要在函数组件中使用 ref，你可以使用 forwardRef（可与 useImperativeHandle 结合使用），或者可以将该组件转化为 class 组件。

## useRef

useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。并且 useRef 可以很方便地保存任何可变值，其类似于在 class 中使用实例字段的方式。

```jsx
function MyComponent() {
  const myRef = useRef(null);
  const onButtonClick = () => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    myRef.current.focus();
  };
  return (
    <>
      <input ref={myRef} type="text" />
      <button onClick={onButtonClick}>聚焦</button>
    </>
  );
}
```

useRef 可以获取 DOM ref
useRef 可以获取最新的值
useRef 内容发生改变并不会通知

## Refs 转发

是否需要将 DOM Refs 暴露给父组件？
在极少数情况下，你可能希望在父组件中引用子节点的 DOM 节点。通常不建议这样做，因为它会打破组件的封装，但它偶尔可用于触发焦点或测量子 DOM 节点的大小或位置。

如何将 ref 暴露给父组件？
如果你使用 16.3 或更高版本的 React, 这种情况下我们推荐使用 ref 转发。Ref 转发使组件可以像暴露自己的 ref 一样暴露子组件的 ref。

什么是 ref 转发？

```jsx
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// 你可以直接获取 DOM button 的 ref：
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

如果在低版本中如何转发？

如果你使用 16.2 或更低版本的 React，或者你需要比 ref 转发更高的灵活性，你可以使用 ref 作为特殊名字的 prop 直接传递。

比如下面这样：

```jsx
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.inputElement = React.createRef();
  }
  render() {
    return <CustomTextInput inputRef={this.inputElement} />;
  }
}
```

以下是对上述示例发生情况的逐步解释：

1. 我们通过调用 React.createRef 创建了一个 React ref 并将其赋值给 ref 变量。
2. 我们通过指定 ref 为 JSX 属性，将其向下传递给 <FancyButton ref={ref}>。
3. React 传递 ref 给 forwardRef 内函数 (props, ref) => ...，作为其第二个参数。
4. 我们向下转发该 ref 参数到 <button ref={ref}>，将其指定为 JSX 属性。
5. 当 ref 挂载完成，ref.current 将指向 <button> DOM 节点。
