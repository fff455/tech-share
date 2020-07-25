# snowpack

snowpack 是一个 bundleless 的构建框架。现阶段 web 构建大型应用还是使用 webpack 的打包方案，使用 webpack 打包在某些方面是存在缺陷的。例如在构建大型 APP 时，webpack 的启动时间会超过 60s，reload 时间也要 10s，20s 以上。这是因为 webpack 在构建时会生成使用 map 存放模块 id 和路径，加载时使用 webpack_require 取出模块进行使用。webpack 等打包工具会在打包时将各个模块放到 bundle 中，项目越大，bundle 文件越大，加载时间越长。

## ESM

snowpack 的原理是在浏览器中使用 ESM。ESM 是 ES 模块在浏览器端的实现，目前主流的浏览器都已经支持。

ESM 例子：

```html
<script type="module">
  import { createApp } from "./main.js";
  createApp();
</script>
```

在 script 标签里设置 type="module"，浏览器会识别添加 type="module"的 script 元素，浏览器会把这段内联 script 或者外链 script 认为是 ECMAScript 模块，浏览器将对其内部的 import 引用发起 http 请求获取模块内容。 在 main.js 里，我们用 named export 导出 createApp 函数，在上面的 script 中能获取到该函数。

浏览器原生支持了文件模块化，这使得本地构建不再需要处理模块化关系并聚合文件。以 ESM 为基础的构建工具有以下几个优点：

1. node_modules 完全不需要参与到构建过程，仅这一点就足以让构建效率提升至少 10 倍。
2. 模块化交给浏览器管理，修改任何组件都只需做单文件编译，时间复杂度永远是 O(1)，reload 时间与项目大小无关。
3. 浏览器完全模块化加载文件，不存在资源重复加载问题，这种原生的 TreeShaking 还可以做到访问文件时再编译，做到单文件级别的按需构建。

## snowpack 对 nodemodules 的处理

可以看到，snowpack 遍历项目源码对 node_modules 的访问，并对 node_modules 进行了 Web 版 install，可以认为 npm install 是将 npm 包安装到了本地，而 snowpack install 是将 node_modules 安装到了 Web API，所以这个命令只需构建一次，node_modules 就变成了可以按需被浏览器加载的静态资源文件。

同时源码中对 npm 包的引用都会转换为对 web_modules 这个静态资源地址的引用：

```jsx
import * as ReactDOM from "react-dom";

// 转换
import * as React from "/web_modules/react.js";
```

但同时可以看到 snowpack 对前端生态的高要求，如果某些包通过 webpack 别名设置了一些 magic 映射，就无法通过文件路径直接映射，所以 snowpack 生态成熟需要一段时间，但模块标准化一定是趋势，不规范的包在未来几年内会逐步被淘汰。

## 开发调试

调试 snowpack dev，编译 snowpack build，会自动以 src/index 作为应用入口进行编译。snowpack dev 命令几乎是零耗时的，因为文件仅会在被浏览器访问时进行按需编译，因此构建速度是理想的最快速。当浏览器访问文件时，snowpack 会将文件做如下转换：

```jsx
// Your Code:
import * as React from "react";
import * as ReactDOM from "react-dom";

// Build Output:
import * as React from "/web_modules/react.js";
import * as ReactDOM from "/web_modules/react-dom.js";
```

目的就是生成一个相对路径，并启动本地服务让浏览器可以访问到这些被 import 的文件。其中 web_modules 是 snowpack 对 node_modules 构建的结果。在这之前也会对 Typescript 文件做 tsc 编译，或者 babel 编译。

## snowpack 劣势

1. 对 ES Modules 的依赖性强，在 npm 上虽然 ES Modules 的包在逐渐增加，但是短期内需要包都需要做额外的处理。例如我想引入 Antd, 发现其中依赖了很多 CommonJS 的模块以及样式未使用 CSS-in-JS, 引入较为繁琐。
2. 对于一些 css，images 资源处理不够友好，需要额外手动处理，并且底层使用 rollup 来进行一次 ES Modules 的导出太过于生硬, 没有强大的自定义的插件或者配置。
3. 太多依赖包会造成网络问题

snowpack 代表的 bundleless 方案肯定是光明的未来，带来的构建提效非常明显，人力充足的前端团队与不需要考虑浏览器兼容性的敏捷小团队都已经开始实践 bundleless 方案了。但对于业务需要兼容各浏览器的大团队来说，目前 bundleless 方案仅可用于开发环境，生产环境还是需要 webpack 打包，因此 webpack 生态还可以继续繁荣几年，直到大的前端团队也抛弃它为止。

目前 snowpack 还不适合用在生产环境。当然用在开发环境还是可以的，但需要承担三个风险：

1. 开发与生产环境构建结果不一致的风险。
2. 项目生态存在非 ESM import 模块化包而导致大量适配成本的风险。
3. 项目存在大量 webpack 插件的 magic 魔法，导致标准化后丢失定制打包逻辑的风险。
