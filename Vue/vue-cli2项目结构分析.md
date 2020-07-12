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
}
```

### webapck配置

webpack配置位于build文件夹下，具体目录如下

```
build
├─build.js
├─check-version.js
├─logo.png
├─utils.js // 工具类方法
├─vue-loader.conf.js   // webpack中vue-loader配置模块
├─webpack.base.conf.js // webpack基础配置
├─webpack.dev.conf.js  // 开发环境webpack配置
└─webpack.prod.conf.js // 生产环境webpack配置
```

#### webpack.base.conf.js

那么从 webpack 基础配置开始入手，开发环境和生产环境的 webpack 配置无非就是根据环境区分后，一同 merge 至基础配置上的。

```javascript
// webpack.base.conf.js
'use strict'
const path = require('path')
const utils = require('./utils')
const config = require('../config')
const vueLoaderConfig = require('./vue-loader.conf')

function resolve (dir) {
  return path.join(__dirname, '..', dir)
}
```

先看开头部分，引入了 node 模块 path，同目录下的 util.js (工具类方法) 与 vue-loader.conf.js (vue-loader配置模块)，以及上文基础配置中提到的 config/index.js。

另外封装了 path.join() 的方法，这边简单提一下path内部的方法与参数：

* path.resolve([from...],to) - 把一个路径或路径片段的序列解析为一个绝对路径。相当于执行cd操作。

* path.join(path1，path2，path3.......) - 将路径片段使用特定的分隔符( window：\ )连接起来形成路径，并规范化生成的路径。若任意一个路径片段类型错误，会报错。若某个片段为 '..'，则会回到目录的上一级。

* __dirname - 当前被执行文件所在的目录。

接下来看正文部分。

```javascript
module.exports = {
  context: path.resolve(__dirname, '../'), // webpack上下文，解析入口的起点，将入口路径设置为build文件夹上一级的项目根目录。由于webpack配置没有放置于根目录下，所以需要增加这个配置，保证后续相对路径的准确。
  entry: { // webpack入口，上文“代码文件构成”部分已经提及，整个vue实例的入口
    app: './src/main.js'
  },
  output: {
    path: config.build.assetsRoot, // 出口文件路径
    filename: '[name].js', // 打包生成的bundle文件名称
    publicPath: process.env.NODE_ENV === 'production' // 外部静态资源路径，根据环境配置加以区别
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: {
    extensions: ['.js', '.vue', '.json'], // 引入js、vue、json文件时不需要扩展名
    alias: { // import或require时的翻译解析
      'vue$': 'vue/dist/vue.esm.js', // import 'vue' 时，指代引入该路径下的js文件
      '@': resolve('src'), // resolve 方法将 'src' 解析为 '根目录/src' ，当前配置下使用 '@' 来方便代码内部互相引用，如引入组件可直接写为 import component from '@components/component'，来指代引入src/components下的组件。
    }
  },
  module: { // 根据扩展名解析文件
    // ...  
  },
  node: {
    // prevent webpack from injecting useless setImmediate polyfill because Vue
    // source contains it (although only uses it if it's native).
    setImmediate: false,
    // prevent webpack from injecting mocks to Node native modules
    // that does not make sense for the client
    dgram: 'empty',
    fs: 'empty',
    net: 'empty',
    tls: 'empty',
    child_process: 'empty'
  }
}
```

在看webpack的module配置之前，先来看一看工具类内的方法 assetsPath 和 vue-loader 的配置内容，配合上文基础配置的内容，我们很容易就能知道assetsPath的作用就是将静态资源放入static文件夹下。

```javascript
// util.js
exports.assetsPath = function (_path) {
  const assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? config.build.assetsSubDirectory
    : config.dev.assetsSubDirectory

  return path.posix.join(assetsSubDirectory, _path)
}
```

```javascript
// vue-loader.conf.js
// ...
module.exports = {
  // scourceMap方面的配置
  loaders: utils.cssLoaders({
    sourceMap: sourceMapEnabled,
    extract: isProduction
  }),
  cssSourceMap: sourceMapEnabled,
  cacheBusting: config.dev.cacheBusting,
  
  transformToRequire: {
    video: ['src', 'poster'],
    source: 'src',
    img: 'src',
    image: 'xlink:href'
  }
}
```

在vue-loader中，有个 transformToRequire 的配置，它节省了组件在引入一些资源时，需要的引入代码，举个例子。

```html
<avatar :src="logoUrl"></avatar>
```

现在有个 <avatar> 控件，我们用原生的 src 来引入其需要的图片，logoUrl 表示路径参数，在没有 transformToRequire 的配置时，我们需要用 require 或者 import 来引入资源。

```javascript
export default {
  data() {
    return{
      logoUrl: require('./asset/logo.png')
    }
  }
}
```

而当在vue-loader中配置了 transformToRequire 的内容后，引入这些资源就可以按如下的写法来写。于是代码就简化了。

```html
<avatar src="./asset/logo.png"></avatar>
```

那么，继续看module部分的配置，就显得非常简单了，使用 vue-loader 解析 vue 文件。其他就是常规配置，包括 babel-loader 解析 js 文件等。图片、视频、字体等资源也用了相应的 loader 进行解析，并打包至static文件夹下。

```javascript
module.exports = {
  // ...
  module: {
    rules: [
      { // 使用vue-loader解析vue文件，具体配置写于vue-loader.conf.js中
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      { // 解析js代码，具体路径为src，test，以及依赖中的js代码
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test'), resolve('node_modules/webpack-dev-server/client')]
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  }
}
```

#### webpack.dev.conf.js

继续看开发环境下 webpack 的配置，首先分析一波引入的模块。

```javascript
'use strict'
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const path = require('path')
const baseWebpackConfig = require('./webpack.base.conf')
const CopyWebpackPlugin = require('copy-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
const portfinder = require('portfinder')
```

都还是一些比较正常的本地配置与外部插件引入。值得提一提的就是几个插件，

* CopyWebpackPlugin : webpack拷贝静态资源插件；

* HtmlWebpackPlugin : 自动生成入口html文件插件，webpack的老熟人插件了；

* FriendlyErrorsPlugin : 上文有提到过，用于调整报错内容;

* portfinder : npm配置端口的依赖。

接下来是主体部分的配置，

```javascript
const HOST = process.env.HOST
const PORT = process.env.PORT && Number(process.env.PORT)

const devWebpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap, usePostCSS: true })
  },
  // cheap-module-eval-source-map is faster for development
  devtool: config.dev.devtool,

  // these devServer options should be customized in /config/index.js
  devServer: {
    // ...
  },
  plugins: [
    // ...
  ]
})
```

HOST 与 POST 先留着，放到下文 devServer 里面说明。总体来看，开发环境的配置就是将开发环境特有的 webpack 配置通过 webpack-merge 合并到 webpack 基础配置当中。里面用到的开发环境特有的静态资源，比如插件 (plugin)，直接在文件内进行了引入。而其需要的自定义设置，其实看注释也可以发现，都是在上文提到的基础配置 (/config/index.js) 中，支持开发人员进行自定义配置的。

那么既然 module 和 devtool 两个部分内容比较少，就先提一提，module.rules 就在 base 配置的基础上，增加了开发环境下不同扩展名样式代码的映射方式，去看工具类(utils)的代码，可以发现样式代码支持的扩展名有 css, postcss, less, sass, scss, stylus, styl 共七种。而 devtool 则是 vue 项目主体源码在控制台的映射方式。

继续看各个部分的细节。

```javascript
const devWebpackConfig = merge(baseWebpackConfig, {
  // ...

  // these devServer options should be customized in /config/index.js
  devServer: {
    clientLogLevel: 'warning', // 控制台日志等级，默认为info，为none时不展示任何日志
    historyApiFallback: { // 任意路径出现404响应的时候会自动转跳至入口html文件
      rewrites: [
        { from: /.*/, to: path.posix.join(config.dev.assetsPublicPath, 'index.html') },
      ],
    },
    hot: true, // 模块热替换
    contentBase: false, // 配合CopyWebpackPlugin插件使用，非禁用时提供静态资源文件路径
    compress: true, // 一起服务都启用gzip压缩
    host: HOST || config.dev.host, // 关于HOST与PORT，其实可以直接在config/index.js内进行配置
    port: PORT || config.dev.port, // 配合上文埋的一个坑，我们知道HOST与PORT也可以通过node环境配置，且优先于config的配置
    open: config.dev.autoOpenBrowser, // 是否自动打开浏览器
    overlay: config.dev.errorOverlay // 在出现编译报错时，是否将报错信息整屏覆盖界面
      ? { warnings: false, errors: true } // 这边还有个人性化的配置，就是只有编译报错才覆盖，不包括warning
      : false,
    publicPath: config.dev.assetsPublicPath, // 静态资源文件目录
    proxy: config.dev.proxyTable,  // 跨域配置
    quiet: true, // 启用后，除启动信息外，其余webpack报错不会出现在控制台
    watchOptions: { // 是否开启代码变化的监听，也可以配置具体数据来作为时间轮询监听
      poll: config.dev.poll,
    }
  },
  plugins: [
    // 将开发环境特有配置信息定义为一个webpack插件
    new webpack.DefinePlugin({
      'process.env': require('../config/dev.env')
    }),
    // 基于webpack-dev-server，开发环境下用于只替换修改部分的代码模块，而不用重新编译。生产环境下HMR不可启用。
    new webpack.HotModuleReplacementPlugin(), 
    // 当开启 HMR 的时候使用该插件会显示模块的相对路径，建议用于开发环境。
    new webpack.NamedModulesPlugin(), // HMR shows correct file names in console on update.
    // 在编译出现错误时，使用 NoEmitOnErrorsPlugin 来跳过输出阶段。这样可以确保输出部分的代码(如console.log)中不会包含错误。
    new webpack.NoEmitOnErrorsPlugin(),
    // 生成项目入口html文件
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    // 迁移静态资源内容
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.dev.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})
```

看完整个细节配置，其实很多内容都在 config/index.js 有所提及，这也侧面反应了 config 才是面向开发者的配置内容。

最后部分则是一个异步方法，获取到端口后开始运行webpack

```javascript
module.exports = new Promise((resolve, reject) => {
  // 在运行 webpack 前先获取到端口信息，可以是node提供，也可以直接在config内配置
  portfinder.basePort = process.env.PORT || config.dev.port
  portfinder.getPort((err, port) => {
    if (err) { // 获取失败，抛出错误
      reject(err)
    } else { // 获取成功，添加入webapck配置
      process.env.PORT = port
      devWebpackConfig.devServer.port = port

      // Add FriendlyErrorsPlugin
      devWebpackConfig.plugins.push(new FriendlyErrorsPlugin({
        compilationSuccessInfo: { // webpack运行完成提示
          messages: [`Your application is running here: http://${devWebpackConfig.devServer.host}:${port}`],
        },
        onErrors: config.dev.notifyOnErrors
        ? utils.createNotifierCallback()
        : undefined
      }))

      resolve(devWebpackConfig)
    }
  })
})
```

#### webpack.prod.conf.js

生产环境下的 ``webpack`` 配置，重点来看其与开发环境的差异

```javascript
'use strict'
const path = require('path')
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')
const CopyWebpackPlugin = require('copy-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')
```

首先还是从引入的模块部分开始入手。显然，像 ``FriendlyErrorsPlugin`` 这类开发环境下的提示性质的插件已经被去除，取而代之的是 ``OptimizeCSSPlugin`` 和 ``UglifyJsPlugin`` 这类的对代码进行优化与压缩的插件，以此来尽可能的减小生产环境下的项目包体。而由于运行环境的差异，生产环境也不再需要运行地址与端口的配置，交由后端进行处理。

```javascript
const webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true,
      usePostCSS: true
    })
  },
  devtool: config.build.productionSourceMap ? config.build.devtool : false,
  output: {
    // ...
  },
  plugins: [
    // ...
  ]
})
```

继续看 ``webpack`` 主体部分的配置，``module`` 部分相比开发环境，多了一个 ``extract: true`` 的配置，查看工具类中的方法，可以发现其实就是启用了 ``vue-style-loader`` 来加载样式代码，依旧是代码的优化。而 ``devtool`` 则不再与开发环境那样使控制台展示源代码，而是经过转译后的js代码，一来可以说是对生产环境下代码的保护，二来也是为了提高加载效率。

```javascript
const webpackConfig = merge(baseWebpackConfig, {
  // ...
  output: {
    path: config.build.assetsRoot,
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
})
```

那么最大的区别，就是开发环境下的 ``devServer`` 替换为了 ``output``，``output`` 的配置就不像``devServer`` 那么繁琐了，仅仅配置了打包输出路径，``bundle`` 文件的命名，以及``chunk``文件的命名。

至于插件部分，一部分已经在开发环境的配置中进行了说明，且这一部分官方的代码注释写的相当详细，所以只挑选了几个值得提一提的生产环境下的插件

```javascript
const webpackConfig = merge(baseWebpackConfig, {
  // ...
  plugin: [
    // ...
    // 非常通用的js代码优化与压缩的插件，在 webpack 章节中也有提及
    new UglifyJsPlugin({
      uglifyOptions: {
        compress: {
          warnings: false
        }
      },
      sourceMap: config.build.productionSourceMap,
      parallel: true
    }),
    // css代码优化与压缩，以及css打包后的命名规则与路径配置
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css'),
      allChunks: true,
    }),
    new OptimizeCSSPlugin({
      cssProcessorOptions: config.build.productionSourceMap
        ? { safe: true, map: { inline: false } }
        : { safe: true }
    }),
    // ...
    // 对项目引入并且用到的依赖进行打包优化
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks (module) {
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
  ]
})
```

最后一块的代码是与我们上文所提及的 ``productionGzip`` 与 ``bundleAnalyzerReport`` 配置相关的。前者规定是否开启gzip，需要配合后端使用。而后者则是用于查看控制台中的 bundle 识图。这两个 if 逻辑在默认的配置下都是不执行的。当然在执行后，我们也可以通过代码发现这两个都会被添加进入 ``webpack.plugins`` 当中。两者的最终目标，依旧是为了优化打包后的代码。

```javascript
if (config.build.productionGzip) {
  const CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}

if (config.build.bundleAnalyzerReport) {
  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}
```

### 项目打包配置

项目打包脚本``build.js``与webpack配置一样位于build目录下。毕竟打包也是基于webpack的。具体打包命令已经在本文开头贴出，从  ``package.json`` 中也可以看出打包命令其实就是 ``node build.js``

由于篇幅比较小，那么直接看代码。

```javascript
'use strict'
require('./check-versions')()

process.env.NODE_ENV = 'production'

// 引入的依赖的作用在下方的注释中具体说明
const ora = require('ora')
const rm = require('rimraf')
const path = require('path')
const chalk = require('chalk')
const webpack = require('webpack')
const config = require('../config')
const webpackConfig = require('./webpack.prod.conf')

// 打包开始提示，使用ora达到一个轮转的loading效果
const spinner = ora('building for production...')
spinner.start()

// rimraf，类似于 rm -rf 命令，删除dist文件夹下的原有打包内容
rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  // 回调方法内进行实际打包操作，当然如果删除的时候出现异常需要抛出
  if (err) throw err
  // 使用webpack进行打包，webpack配置来自于生产环境webpack配置webpack.prod.conf.js
  webpack(webpackConfig, (err, stats) => {
    // 打包完成后的回调
    spinner.stop() // 在终端结束轮转动画表示打包结束
    if (err) throw err // 遇到报错抛出异常
    // 输出打包内容，process.stdout.write类似于console.log，它会将打包输出的内容进行格式化
    process.stdout.write(stats.toString({ // 输出内容格式化配置
      colors: true, // 打包内容颜色高亮
      modules: false, // 去掉内置模块信息
      children: false, // 去掉子模块，如果需要编译typeScript，则置为true
      chunks: false, // 增加依赖信息，设置false能允许较少的冗长输出
      chunkModules: false // 去除包里内置模块的信息
    }) + '\n\n')

    if (stats.hasErrors()) { // 如果上方有错误抛出，就进入打包错误的逻辑
      console.log(chalk.red('  Build failed with errors.\n'))
      process.exit(1)
    }

    // 无错误抛出，输出打包完成提示，chalk依赖就用于在终端中输出带颜色的提示
    console.log(chalk.cyan('  Build complete.\n'))
    console.log(chalk.yellow(
      '  Tip: built files are meant to be served over an HTTP server.\n' +
      '  Opening index.html over file:// won\'t work.\n'
    ))
  })
})
```

### 其他配置

其他还有一些位于根目录下的插件配置，包括每个项目都有的 ``.gitignore`` 与 ``README.md``，以及 ``package.json``。在依赖配置文件中，我们可以看到 vue-cli 一共提供了三个命令，两个执行功能，分别用于开发环境与生产环境。

```json
{
  "scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "build": "node build/build.js"
  },
}
```

而另外在根目录下并不是很常见的配置，就 vue-cli 使用的一些非常便利的开源插件

* .babelrc - 用于设置 ``javascript`` 的编译规则以及插件

* .postcssrc.js - 方便样式代码编写，如插件 ``autoPreFixer`` 提供的自动补充浏览器前缀的功能，减少了许多样式代码编写的工作量

* .eslintrc.js - eslint是用来管理和检测js代码风格的工具，可以和编辑器搭配使用，如vscode的eslint插件 当有不符合配置文件内容的代码出现就会报错或者警告。如果项目建立配置时不选择开启eslint，那么根目录下也不会有这个配置文件

### 结语

前后一共分了六期对``vue-cli2``脚手架代码进行了分析。总的来说，``vue-cli``作为一个``Vue``官方提供的前端脚手架，对于一个新人上路的Vue开发来说是相当友好的，确实是做到了低成本，上手即开发的效果。如果有更复杂的需要，也可以基于``vue-cli``脚手架进行改进，更加符合自身的项目要求。在``Vue``已经发展到3.x的当下，虽然``vue-cli2``已经有了``vue-cli3``这一全新版本，但它依旧被不少开发所喜爱。

对于``vue-cli2``的构建思路，其本质上还是在于``webpack``与``Vue`的相结合。在学习过程中，可以明显感受到，大部分时间其实都是在学webpack配置，在一个个搞懂webpack代码中各个纷繁复杂配置的作用。甚至可以把vue-cli当成一个学习webpack的扩展。
