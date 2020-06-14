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

### 基础配置

Vue入口与基本的代码结构清楚之后，再来看config目录下的基础配置，文件目录如下：

```
config
├─dev.env.js  // 开发环境配置
├─index.js    // 基础配置
└─prod.env.js // 生产环境配置
```

开发环境与生产环境的配置其实就是在两个环境下设置不同node的全局变量来区分两个环境

```javascript
// prod.env.js
'use strict'
module.exports = {
  NODE_ENV: '"production"'
}
```

```javascript
// dev.env.js
'use strict'
const merge = require('webpack-merge')
const prodEnv = require('./prod.env')

module.exports = merge(prodEnv, {
  NODE_ENV: '"development"'
})
```

那么看了代码后很容易就有一个疑问，为什么开发环境要多此一举地用 webapck-merge，直接和生产环境一样的写法不是更好？

* 其实光看默认给的代码，确实是多此一举，但真正在开发的时候，两个环境下必然存在其他的配置，使用 webpack-merge 后，在生产环境添加的配置，就不用在开发环境再添加一遍，节省了时间，提高了容错率。而开发环境特有的配置只需在开发环境的配置中添加即可。

* 当然有个问题是这个写法没法为生产环境提供特有的配置，但是在实际生产中，其实生产环境需要的配置往往在开发环境下也是必要的，这也是为什么要把merge加在开发环境下的原因。

接下来是基础配置下的 config/index.js 文件，所谓的基础配置，其实也是webpack中的部分配置和打包脚本的部分配置。config/index.js 文件中的内容，更倾向于面向开发人员的个性配置。那么直接来看代码。

```javascript
// config/index.js
const path = require('path')

module.exports = {
  dev: {
    // ...
  },
  build: {
    // ...
  }
}
```

整体来看，就配置了 dev 与 build ，也就是上文说的 webpack 开发环境部分和打包脚本部分的面向开发的配置。其实从引入角度也可以看出来。

![vuecliconfigdev](./image/vuecliconfigdev.png)

dev 的配置重点在 webapck.dev.conf.js 内进行了使用，下文也会提及。而在其他文件中的使用，就都是配合条件运算符(?:)区分环境来使用的。再看dev 部分的代码。

```javascript
module.exports = {
  dev: {
    // Paths
    assetsSubDirectory: 'static', // 静态资源存放路径
    assetsPublicPath: '/', // 根目录下存放静态资源
    proxyTable: {}, // 跨域配置

    // Various Dev Server settings
    host: 'localhost', // 本地运行地址，可用process.env.HOST代替
    port: 8080, // 本地运行端口，可用process.env.PORT代替
    autoOpenBrowser: false, // 是否运行代码后自动打开浏览器
    errorOverlay: true, // 是否在控制台显示报错
    notifyOnErrors: true, // 是否使用FriendlyErrorsPlugin调整报错内容
    poll: false, // 是否开启代码变化的监听，也可以配置具体数据来作为时间轮询监听
    
    // Source Maps
    devtool: 'cheap-module-eval-source-map', // 控制台中显示的代码内容，不同的source-map会显示不同的代码（如打包前，打包后的代码），且构建速度各有区别。开发环境下使用的这个source-map有助于调试。
    cacheBusting: true, // 个人理解是在代码改变后用于改变控制台代码的缓存，方便开发调试
    cssSourceMap: true // 是否开启css代码转换，设为false后将无法在控制台定位样式代码的来源，所以开发环境下最好为true
  },
  build: {
    // ...
  }
}
```

![vuecliconfigbuild](./image/vuecliconfigbuild.png)

build 则重点在 webpack.prod.conf.js 内进行了使用，而其他地方的使用其实都是在条件应算符下与dev相对应的选择。所以打包的配置同时也可以说是生产环境的配置。

```javascript
module.exports = {
  dev: {
    // ...
  },
  build: {
    // Template for index.html
    index: path.resolve(__dirname, '../dist/index.html'), // 打包后index.html的位置

    // Paths
    assetsRoot: path.resolve(__dirname, '../dist'), // 打包后的静态资源位置
    assetsSubDirectory: 'static', // 打包后静态资源文件夹 
    assetsPublicPath: '/', // 静态资源相对index.html的位置，'/'表示绝对路径。打包后在本地打开index.html会出现页面静态资源加载不出的情况，改为'./'可解决，而运行至服务器的代码需要保持'/'的设置，当然有特殊情况其他再论。

    // Source Maps
    productionSourceMap: true, // 控制打包后的js、css文件是否生成map文件
    devtool: '#source-map', // 代码转换配置，控制台将显示打包后的代码

    productionGzip: false, // 是否开启gzip，需要配合后端使用，若开启，需要先安装依赖compression-webpack-plugin
    productionGzipExtensions: ['js', 'css'], // gzip支持的文件类型

    bundleAnalyzerReport: process.env.npm_config_report // 能够在浏览器中看到bundle的分析图
}
```

### webapck配置

webpack配置位于build文件夹下，具体目录如下

```
build
├─build.js
├─check-version.js
├─logo.png
├─utils.js
├─vue-loader.conf.js   // webpack中vue-loader配置模块
├─webpack.base.conf.js // webpack基础配置
├─webpack.dev.conf.js  // 开发环境webpack配置
└─webpack.prod.conf.js // 生产环境webpack配置
```


### 项目打包配置

### 环境配置

### 其他配置