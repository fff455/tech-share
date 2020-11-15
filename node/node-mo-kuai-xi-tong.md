# Node 模块系统

## 背景

像 Python 或者 Java 来编写应用程序时，首先它们拥有非常多的库，在开发时的效率能提高，对于文件 IO 和调用 OS 级别的操作都有标准的接口可用。对于 JavaScript 来说，它本身的规范非常弱，有以下几个缺陷

* 没有模块
* 没有标准库
* 没有标准接口
* 没有包管理

针对 JavaScript 的薄弱规范，Node 借鉴 CommonJS 实现了模块系统

## CommonJS

1. 模块引用

```javascript
const math = require("math");
```

1. 模块定义

```javascript
const add = (a, b) => a + b;
exports.add = add;
```

1. 模块标识

在进行模块引用时，`const math = require('xxx')`，`xxx`就是这个模块的标识，它可以是一个字符串，也可以是一个路径。每个模块拥有独立的空间，互不干扰，不需要考虑变量污染和命名空间

## Node 模块

Node 中的模块包括两类：Node 核心模块（path/fs/...）和文件模块（用户编写的模块和通过包管理安装的模块）

### Node 模块引用/导出

模块引用与 CommonJS 相同，对于模块导出，一个模块中有一个`exports`对象用于导出当前模块的方法或者变量

* exports

```javascript
const add = (a, b) => a + b;
exports.add = add;
```

* module.exports

```javascript
const add = (a, b) => a + b;
module.exports = {
  add
};
```

在 Node 中，既可以使用`exports`，也可以使用`module.exports`来导出模块，这两种方法有区别。

对于`module.exports`来说，它指向一个对象，初始化指向`{}`，你可以改变它的指向，例如上文，指向修改为一个新的对象，对象中有属性`add`。

但是对于`exports`来说，它指向的是`module.exports`，实际上是 Node 提供了一个变量来方便访问`module.exports`。通过`exports.xxx = xxx`我们就可以对`module.exports`的属性进行修改，导出时依旧导出`module.exports`。所以！如果直接对`exports`进行修改，是不会有效果的。例如`exports = {add: '123'};`，它直接修改了`exports`变量的指向，从`module.exports`变成指向一个新对象，但是这样不会修改`module.exports`，所以导出时，还是导出了`module.exports`，新对象`{add: '123'}`并不会导出。

### Node 模块实现

在 Node 中引用模块，会经历以下一个步骤：缓存-路径分析-文件定位-编译执行

1. 缓存

如果模块在此前已经编译执行过，Node 会缓存编译执行后的结果，当再次引用时，路径分析-文件定位-编译执行这几个步骤都不会执行

1. 路径分析

当缓存未命中时，会进行路径分析。路径分析会根据模块标识的分类进行不同方法的查找，模块标识大概可以分为以下三类：

* 核心模块

核心模块在 Node 源代码编译过程中已经被编译成二进制代码，当路径匹配到核心模块时，文件定位-编译执行这几个步骤不会执行，直接使用 Node 编译后的二进制文件

* 以`./`或者`/`开始的路径形式的文件模块

当路径分析未匹配到核心模块，但是匹配到路径形式的文件模块时，Node 会先将路径转化成绝对路径，定位文件并编译执行，把绝对路径作为 key 和编译执行的结果存入缓存

* 自定义模块，例如 npm 安装的模块

当路径分析都未匹配到核心模块和路径形式的文件模块时，Node 认为它是一个自定义模块，然后根据`module.paths`进行查找。`module.paths`是模块中的一个字符串数组，它存储了当前模块的模块路径。例如，我们有一个文件，它的路径是`/a/b/c/test.js`，它的`module.paths`的值是

```javascript
[
  "/a/b/c/node_modules",
  "/a/b/node_modules",
  "/a/node_modules",
  "/node_modules"
];
```

所以，根据`module.paths`进行查找时，会先查找当前路径的下的`node_modules`文件夹，并进行文件定位，如果没有定位到，则去寻找父目录的`node_modules`文件夹进行文件定位，直至找到根目录的`node_modules`文件夹进行文件定位。如果直至根目录的`node_modules`文件夹都没有定位到文件，则抛出查找失败的错误。

1. 文件定位

当匹配到路径形式的文件模块或者自定义模块时，Node 会去定位文件。在写模块标识符时，可以不写后缀名，这就依赖于 Node 文件定位的功能。  
Node 文件定位会按照顺序先去寻找当前路径加扩展名是`.js`/`.json`/`.node`的文件。如果存在这个文件，就返回这个文件并对这个文件进行编译。  
如果这三个扩展名都没有匹配成功，Node 认为这个路径是一个文件夹，然后 Node 去这个文件下按照顺序去寻找文件名是`index.js`/`index.json`/`index.node`的文件。如果存在这个文件，就返回这个文件并对这个文件进行编译。

1. 编译和执行

当进行编译和执行时，文件类型有 3 种情况`.js`/`.json`/`.node`，根据这三种情况，Node 编译的策略也不同

* js 文件

> 我们在 Node 中可以使用`require`/`exports`/`module`/`__filename`/`__dirname`就是因为在编译执行的时候 Node 将这些变量传了进来

首先使用 Node 的`fs`模块读取文件内容，然后用一个函数来包裹：

```javascript
(function(exports, require, module, __filename, __dirname) {
  // 文件内容
});
```

通过这样实现了不同模块直接的作用域隔离，并传入了 CommonJS 模块规范以及文件名和所在文件夹名的变量。

* node 文件

`.node`文件是通过 C/C++写的扩展文件，将模块的 exports 和扩展文件进行关联，它通过 Node 的`process.dlopen()`进行打开和执行。由于是使用 C/C++写的，所以不需要编译阶段，它的执行效率相对更高。  
在 Windows 和\*nix 系统下，`dlopen()`的实现方法不同，主要使用了`libuv`进行了封装

* json 文件

首先通过 Node 的`fs`模块读取文件内容，然后通过

```javascript
JSON.parse(/** 文件内容 **/);
```

得到结果，赋值给`module.exports`

