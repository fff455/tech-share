# TypeScript

## 重写类型的动态查找

在你的项目里，你可以通过 `declare module 'somePath'` 来声明一个全局模块的方式，用来解决查找模块路径的问题：

```text
// globals.d.ts
declare module "foo" {
  // some variable declarations export var bar: number;
}
```

接着：

```text
// anyOtherTsFileInYourProject.ts
import * as foo from "foo";
// TypeScript 将假设（在没有做其他查找的情况下）
// foo 是 { bar: number }
```

## `import/require`  仅仅是导入类型

以下导入语法：

```text
import foo = require("foo");
```

它实际上只做了两件事：

* 导入 foo 模块的所有类型信息；
* 确定 foo 模块运行时的依赖关系。

## 命名空间 namespace

在确保创建的变量不会泄漏至全局变量中时，这种方式在 JavaScript 中很常见。当使用基于文件模块时，你无须担心这点，但是此种方式，仍然适用于合理的函数逻辑分组中。因此 TypeScript 提供了 `namespace` 关键字用来描述这种分组，如下所示：

```text
namespace Utility {
  export function log(msg) {
    console.log(msg);
  }
  export function error(msg) {
    console.log(msg);
  }
}
// usage
Utility.log("Call me");
Utility.error("maybe");
```

`namespace` 关键字通过 TypeScript 编译后，与我们看到的 JavaScript 代码一样：

```text
(function (Utility) {
    // 添加属性至 Utility
})(Utility || Utility = {});
```

有一点值得注意的是，命名空间是支持嵌套的。因此，你可以做一些类似于在 `Utility` 命名空间下嵌套一个命名空间 `Messaging` 的事情。

对大多数项目来说，我们推荐使用一个使用 `namespace` 的外部的模块，用来快速的演示和移植旧的 JavaScript 代码。

## TypeScript 是怎么确定单个断言是否足够

当 `S` 类型是 `T` 类型的子集，或者 `T` 类型是 `S` 类型的子集时，`S` 能被成功断言成 `T`。这是为了在进行类型断言时提供额外的安全性，完全毫无根据的断言是危险的，如果你想这么做，你可以使用 `any`。

