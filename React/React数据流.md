# React 数据流

React 的核心思想是`UI=render(data)`，所以数据对于 React 的组件来说是非常重要的，因为它直接关系着 UI 如何展现。

## React 原生数据流

React 最基本的数据有两种：State 和 Props。State 是组件内部的状态数据，而 Props 是从外部传入的参数。我们用一个例子来看一下这两者是如何使用的。

```jsx
class A extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      counter: 0,
    };
  }

  render() {
    const { title } = this.props;
    const { counter } = this.state;
    return (
      <div>
        <h2>{title}</h2>
        Counter: {counter}
        <button
          onClick={() => {
            this.setState({ counter: counter + 1 });
          }}
        >
          +
        </button>
      </div>
    );
  }
}
```

我们在组件 A 中使用了 State 和 Props，State 是 A 内部维护的状态，可以使用 this.setState 来对状态进行更新；Props 是调用 A 时传入的，我们可以这么来使用：

```jsx
class B extends React.Component {
  render() {
    return (
      <div>
        <A title="my first counter" />
        <A title="my second counter" />
      </div>
    );
  }
}
```

这里面蕴含了 React 做的很好的组件化思想，B 组件中调用 A 时，只需要关心双方之间的接口（Props）传值即可，不需要关心 A 组件内部的状态如何变化。

由此我们可以看到，在组件内部，使用 State 作为数据流，State 改变时，组件的 UI 也会改变。如果涉及到父子组件，那么父向子的传递可以使用 Props 来进行实现。当子组件的 Props 或者 State 改变时，都会重新渲染。

在使用一个组件时，如果它的渲染完全由 Props 决定，也就是内部没有 State，这样的组件称作一个纯组件。实际使用上，每个 React 组件都可能包含了 Props 和 State，这样其实时不利于组件的管理，因为每个组件内部都有各自的 State，在查问题时可能会对理解/定位上造成困难。一种思想是将 State 向父组件抛，以上述例子，我们可以只将 A 实现为一个纯组件，它接受一个 Props 中 count 的数值，然后展示出来，当 A 需要更新时，调用 Props 中的回调函数，通知父组件进行更新：

```jsx
class A extends React.Component {
  render() {
    const { title, counter, updateCounter } = this.props;
    return (
      <div>
        <h2>{title}</h2>
        Counter: {counter}
        <button
          onClick={() => {
            updateCounter(counter + 1);
          }}
        >
          +
        </button>
      </div>
    );
  }
}
```

```jsx
class B extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      counter1: {
        title: "my first counter",
        counter: 0,
      },
    };
  }
  render() {
    const { counter1 } = this.state;
    return (
      <div>
        <A
          title={counter1.title}
          counter={counter1.counter}
          updateCounter={(c) => {
            this.setState({
              counter1: { ...counter1, counter: c },
            });
          }}
        />
      </div>
    );
  }
}
```

这样也给兄弟组件之间传值提供了一种可行的方式：可以先将数据向上提到公共父节点，通过父节点来处理/同步各自的数据，子组件可以调用回调函数来调用更新数据。

然后我们来讨论下更加复杂的情况：祖父节点和子节点应该如何传递数据。假如祖父节点和子节点之间跨越的层级不多，比如 3 或 4 层，那么其实可以将 Props 依次传递下去。但是如果层级变得更多时，使用 Props 逐层传递会很费时费力。针对这种场景，可以使用 React 的 Content API 进行处理。

```jsx
// 多层数据传递 Props处理
class C extends React.Component {
  render() {
    const sameTitle = "same title";
    return (
      <div>
        <B title={sameTitle} />
      </div>
    );
  }
}

class B extends React.Component {
  render() {
    const { title } = this.props;
    return (
      <div>
        <A title={title} counter={0} />
        <A title={title} counter={100} />
      </div>
    );
  }
}

class A extends React.Component {
  render() {
    const { title, counter } = this.props;
    return (
      <div>
        <h2>{title}</h2>
        Counter: {counter}
        <button>+</button>
      </div>
    );
  }
}
```

```jsx
// 多层数据传递 Content处理
const TitleContext = createContext();
class C extends React.Component {
  render() {
    const sameTitle = "same title";
    return (
      <div>
        <TitleContext.Provider value={sameTitle}>
          <B />
        </TitleContext.Provider>
      </div>
    );
  }
}

class B extends React.Component {
  render() {
    return (
      <div>
        <A counter={0} />
        <A counter={100} />
      </div>
    );
  }
}

class A extends React.Component {
  render() {
    const { counter } = this.props;
    return (
      <div>
        <TitleContext.Consumer>
          {(
            context // context 为 Provider 的 value
          ) => <h2>{context}</h2>}
        </TitleContext.Consumer>
        Counter: {counter}
        <button>+</button>
      </div>
    );
  }
}
```
