# Vue2.6.x源码阅读

## 编译

* 获取[Vue2.6.x源码](https://github.com/vuejs/vue)，dev分支即为最新源码，与去年最新发布的[Vue3.x](https://github.com/vuejs/vue-next)做好区分。获取后进行编译并运行。

  ```shell
  # 获取源码
  $ git clone git@github.com:vuejs/vue.git

  # 安装依赖
  $ yarn

  # 代码运行
  $ yarn dev
  ```

* 编译完成后，最新编译获得的``Vue``开发版本(development)的代码会自动生成于``~/dist/vue.js``。当前端项目引入这一份生成的代码并创建一个``Vue``实例后，那它就已经成为了一个``Vue``项目。

## 调试

* 在根目录下存在一个样例文件夹(``~/examples``)，供``Vue``的学习者阅读使用。当然也可作为源码阅读时的demo用于调试``Vue``源码。

* 当前``~/examples``目录下共有11份简单的样例工程，每个工程的入口文件``index.html``都引入了``vue.min.js``(生产环境版本)，我们可将其替换为开发版本的``vue.js``以对源码打断点或进行调试。

  ![](./image/vuemin.png)

* 样例修改完引入的文件后，此时在开发工具中看到的依然为源码经过编译后的代码。可通过更改编译脚本配置，以及``sourcemap``模式重新运行代码。

  ```javascript
  // ~/scripts/config.js
  function genConfig (name) {
    const opts = builds[name]
    const config = {
      input: opts.entry,
      external: opts.external,
      // 增加sourceMap配置
      sourceMap: true, 
      // ...
    }
  }
  // ...
  ```

  ```shell
  # 启用sourcemap
  $ yarn dev --sourcemap
  ```

* 完成后便可以在``devtool``中看到源码内容。

  ![](./image/vuesourcemap.png)

## 目录结构

### 根目录

* **benchmarks**: 复杂情况下的Vue样例，如大量数据表格、服务端渲染(ssr)、渲染大量svg图片。

* **dist**: 构建后文件的输出目录，如``vue.js``、``vue.min.js``。

* **examples**: ``Vue``应用demo，统一引用了输出目录下的``vue.min.js``。

* **flow**: 类型声明，定义了源码中所使用的各种类型，包括``VNode``、``GlobalAPI``等，使用开源项目 [Flow](https://flowtype.org/)。

* **packages**: ``Vue``相关的一些依赖，如``Weex``，可直接引入使用。

* **scripts**: 用于存放一些``npm``脚本，配合``webpack``与``rollup``等工具对源码进行编译、测试、构建打包等。上文增加的sourcemap配置就位于``~/scripts/config.js``中。

* <strong style="color:red">src</strong>: Vue核心源码，学习的重点。

* **test**: 测试用例目录，包含了单元测试unit、e2e测试(用户真实场景)、服务端渲染(ssr)和weex的一些测试用例。通过``yarn test``执行。

* **types**： ``Vue2.6.x``已经能够支持开发者使用``typescript``，该目录定义了``typescript``类型声明文件。

* 其他文件：

  * 代码规范配置文件： ``.editorconfig``、``.eslintrc.js``、``eslintignore``

  * 类型检查配置文件: ``.flowonfig``

### 核心代码

