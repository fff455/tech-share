## `$0` / `$1` / `$2` ...

在 element 中单击一个节点，在 console 中输入$0 即可显示选中的节点,$1 则表示在\$0 之前选择的元素，以此类推

## `$_`

使用`$_`可以访问最近一次 `console` 输出的内容

## `$() === document.querySelector()` / `$\$() === querySelectorAll()`

## `copy()`

通过 copy(xxx) 方法在控制台里复制你想要的东西，例如

```jaascript
copy($_)
```

在 Network 标签下的所有的 xhr 请求,都可以右击选择`copy as Fetch`复制为一个完整的 Fetch 请求的代码

## `console.dir`

`console.dir`可以将 element 节点以对象的形式输出

## `console.table`

`console.table`可以将对象的节点和值以 table 形式两列的形式展开输出

## `console.time`

示例：

```javascript
const timeUnique = "test timer";
console.time(timeUnique);
Array(1000)
  .fill(0)
  .map((e) => e + 1);
console.timeEnd(timeUnique);
```

输出`test timer: 0.10693359375ms`

## `console.profile`

示例：

```javascript
const profileUnique = "test profile";
console.profile(profileUnique);
Array(1000)
  .fill(0)
  .map((e) => e + 1);
console.profileEnd(profileUnique);
```

打开 Javascript Profile 即可看到`test profile`的详细性能信息

## `console.count`

示例：

```javascript
const countUnique = "test count";
Array(10)
  .fill(0)
  .forEach(() => {
    console.count(countUnique);
  });
```

输出

```
test count: 1
test count: 2
test count: 3
test count: 4
test count: 5
test count: 6
test count: 7
test count: 8
test count: 9
test count: 10
```

## `clear`

使用`clear()`/`console.clear()`可以清除 console 内容

## `async` / `await`

在 console 中可以直接使用 await，不需要 async 包裹

## `debugger`

在源码中写`debugger`可以定义一个断点，能够直接在 source 中运行到此断点并暂停
