# vue-cli2.x 的使用与项目结构分析

> vue-cli是vue官方提供的一个前端脚手架，专门用于快速构建一个单页面web应用，自动生成基于vue + webpack的项目模板。

## 基本使用

### 引入vue-cli

* 全局引入vue-cli2.x

  ```shell
  npm install -g vue-cli
  ```

* 引入完成检测

  ```shell
  vue --version
  ```

* 注：区别于vue-cli3.x的引入方法

  ```shell
  npm install -g @vue/cli
  ```

### 创建项目

* 在所需目录下，创建一个基于vue + webpack的前端项目

  ```shell
  vue init webpack <project-name>
  ```

* 根据提示配置信息后创建完成

  ![vueclidemo](./image/vueclidemo.png)

## 代码结构

### 代码目录

```
vue-cli-demo
├─ build         // 打包与webpack配置
├─ config        // 开发环境与生产环境配置
├─ node_modules  // 依赖包
├─ src           // 代码文件
├─ static        // 静态文件
├─ .babelrc      // babel编译器配置
├─ .editconfig   // 项目统一代码风格配置
├─ .gitignore
├─ .postcssrc.js // 浏览器兼容配置
├─ index.html    // 入口文档
├─ package-lock.json
├─ package.json
└─ README.md     // 项目说明
```

### 代码文件构成

```
src
├─assets     // 图片资源
├─components // Vue页面/组件
├─router     // vue-router配置
├─App.vue
└─main.js
```

```javascript
// main.js
import Vue from 'vue'
import App from './App' // 引入入口组件App
import router from './router' // 引入vue-router配置

Vue.config.productionTip = false // 关闭开发环境下的官方提示

new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
```

首先来看main.js的内容，这也是整个项目代码部分的入口。其功能，就是创建了一个Vue实例，并且引入了名为App的入口组件。这也是整个项目唯一一次创建Vue实例。在这点上可以有些理解：

* 作为一个单页面应用的脚手架，项目构建的页面就只有一个Vue实例。若在浏览器或者客户端下打开新的页面，则两个页面属于两个不同的实例。

* Vue文件本身就是一个实例，但同时也可以被注册成为Vue组件(component)被其他组件所引入，而作为组件引入时，不将其视为一个实例看待。

* vue-cli的Vue实例挂载在id为app的dom上，这一点我们也可以在入口文档index.html中看到。

  ```html
  <!-- index.html -->
  <!DOCTYPE html>
  <html>
    <!-- ... -->
    <body>
      <div id="app"></div>
    </body>
  </html>
  ```

* 实例引入的App组件，相当于一个根节点，通过router将不同界面串在一起，构成一棵树。不同界面下也可以引入新的组件，组件之间可以进行通信等操作。

那么再来看一下main.js中反复提及的App组件与router:

```html
<!-- App.vue -->
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view/>
  </div>
</template>
```

App.vue其实就是一个正常的Vue文件，上述代码给出了代码模板的部分，vue-cli作为官方脚手架很自然的引入了一个Vue的logo，当然这肯定得直接去掉。然后就是router标签，可以在上文main.js里看到，在Vue实例创建的时候就全局挂载了router，所以App.vue不再需要引入什么文件就可以直接使用router，这与Vuex同理。

```javascript
// router/index.js
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    }
  ]
})
```

router部分则就是vue-router的引入、使用，以及注册在router上的Vue组件的配置，vue-cli默认会注册一个HelloWorld组件，我们可以在components下面找到，这个HelloWorld也算是官方给出的一个Vue文件的标准写法。

### webapck配置

### 项目打包配置

### 环境配置

### 其他配置