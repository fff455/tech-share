# Vue2.x全局挂载整理

> 基于``Vue2.x``与``@vue/cli``脚手架实现组件、工具类方法、混入、指令与样式常量的全局挂载

## 前言

* 全局挂载对于开发人员来说，是一个友好且非常方便的开发模式，在工程内合理地使用全局挂载能够有效的提高开发效率。

* 在Vue工程中使用全局挂载的同时，也是对工程内组件、工具类、混入等各类代码的整理与归类。使得目录结构更为明了。

* 随着产品开发的推进，不同类型内容的全局挂载量也会逐渐膨胀。产品组工作流程与组织架构也会随之调整，也因此对于全局挂载的内容必须要做好文档维护与同步。以保证新人能够迅速且正确上手，也保证不同开发之间做好沟通，防止项目组内的出现重复工作。

* 对于全局挂载的使用，需要根据产品本身架构、项目迭代周期长短、工程代码量大小等实际情况进行评估。一般来说，``Vue``框架多应用于单页面应用。全局挂载内容的增加无疑会逐渐加大首次打开页面的性能压力，资源包体会逐渐增大。所以如果对性能有比较高的要求，可以将全局挂载的代码进行评估，若使用范围比较小，按需引入将是更好的选择。另外，如果产品本身就是多页面应用，也同样更适合减少全局挂载内容量，在开发效率与界面性能之间做一个权衡后再作决定。

## 组件(components)

* 组件的全局挂载是最常用的挂载项，对于单页面应用来说，全局挂载组件一来能够提高开发效率，无需重复多次在界面内引用组件库。同时也能在首页加载时一次性加载到组件库的资源包，并渲染其中需要使用的部分。

* 也如上文前言所述，全局的组件挂载需要考虑到某些类型的组件库资源包会比较大，而且在对首页性能要求较高的情况下，或者本身并非单页面应用的情况下。减少全局挂载组件量，且按需``import``会是一个更折衷合理的方案。亦或是在业务代码中异步加载其中非首现的组件资源，比如弹窗、标签页等。

* 全局混入挂载配合``Vue``的``component()``api实现

  ```javascript
  // components/index.js
  import compA from "./compA"
  export default {
    compA,
    compB: () => import(/* webpackChunkName: "components/compB" */ './compB') // 异步引入
  }
  ```

  ```javascript
  // main.js
  import components from './components'
  import Vue from 'vue'
  Vue.use({
    install(Vue) {
      for(const key in components) {
        Vue.component(key, components[key])
      }
    }
  })
  ```

* 挂载第三方组件库也同理使用``use()``方法，不过主流的组件库多对组件资源进行了拆分，能够做到引入局部组件资源并挂载在我们自己的``Vue``实例上，以``ant-design-vue``为例

  ```javascript
  import Vue from 'vue';
  import { Button, message } from 'ant-design-vue';

  // 全局挂载按钮与弹窗
  Vue.component(Button.name, Button);
  Vue.component(Button.Group.name, Button.Group);
  Vue.use(Button);

  Vue.prototype.$message = message;
  ```

* 除去最基本的自定义组件与第三方组件库的挂载，再想象一下这样一个开发需求场景，如果对于某个文件夹下的自定义组件库都需要进行全局挂载，而又不想每新建一个都需要再次在入口文件中进行引入。则可以通过一下方法实现这个自动挂载功能。

  ```javascript
  import Vue from 'vue'
  import upperFirst from 'lodash/upperFirst'
  import camelCase from 'lodash/camelCase'

  // 匹配components目录(包括子目录)下的所有vue文件
  const requireComponent = require.context('../components', true, /\w+\.vue$/)

  requireComponent.keys().forEach(fileName => {
    // 获取所有的组件与文件名
    const componentConfig = requireComponent(fileName)
    const componentName = upperFirst(
      camelCase(
        // 获取和目录深度无关的文件名
        fileName
          .split('/')
          .pop()
          .replace(/\.\w+$/, '')
      )
    )

    // 全局挂载组件
    Vue.component(
      componentName,
      // 如果这个组件选项是通过 `export default` 导出的，那么就会优先使用前者。
      componentConfig.default || componentConfig
    )
  })
  ```

## 工具类方法(utils)

* 工具类方法说的通俗点就是全局函数，在上文组件的全局挂载中其实已经涉及到了全局函数的挂载方式。弹窗本身为一个函数式的组件，在实例上完成全局挂载后，即可直接调用``this.$message()``这个方法使用弹窗组件。其余工具类方法也可以通过同样的方式实现挂载。

  ```javascript
  // main.js
  Vue.prototype.$message = message;
  ```

* 实现的本质，就是在``Vue``实例(对象)的原型对象上注册全局方法，而对于业务代码而言，其中的``this``均指向``Vue``实例本身，所以可以直接通过``this.$apiName()``的形式调用。一般常用的全局挂载方法有如深拷贝(deepCopy)、精确的数字计算方法、封装过的请求方法等。

## 指令(directives)

* 关于指令已在[Vue2.x指令](https://github.com/fff455/tech-share/blob/master/Vue/Vue2.x%E6%8C%87%E4%BB%A4.md)一文中有过详细的使用介绍，其中也包括了指令的全局挂载，这里不过多赘述。

  ```javascript
  // main.js
  // 注册一个全局自定义指令 `v-focus`
  Vue.directive('focus', {
    // 当被绑定的元素插入到 DOM 中时……
    inserted: function (el) {
      // 聚焦元素
      el.focus()
    }
  })
  ```

## 混入(mixins)

* 在Vue项目中，混入相对来说是一个具有争议的功能，理想情况下，混入可以是通用业务功能，也可以是通用的工具方法，合理使用混入能够有效减少不必要的代码量。

* 但如果在缺乏使用经验情况下使用，混入就会在代码迭代与增加中逐渐趋于混乱，繁杂的混入文件一来会大大提高开发人员的学习成本，二来各个业务界面的使用需求也是各不相同，容易产生大量冗余的代码。一份混入文件会因为归类不清而不断膨胀，也可能因为过于在意分类而使文件数量过多。

* 非全局的混入已是如此，全局混入的使用更需要谨慎处理。一般使用于优化整个工程的代码当中，比如按需引入vuex，就需要配合全局混入功能实现。

* 全局混入挂载配合``Vue``的``install()``与``use()``实现

  ```js
  // global-mixins.js
  module.exports = {
    // install方法用于全局注册
    install: (Vue) => {
      // 全局混入
      Vue.mixin({
        // 入参为Vue实例对象
        data() {
          return {
            // 混入内的参数
            key: 'value'
          }
        }
        methods: {
          // 全局混入的方法
        }
        beforeCreate() {
          // 全局混入中对于某个生命周期进行处理
        },
      });
    },
  };
  ```

  ```javascript
  // main.js
  import mixins from './global-mixins.js';
  Vue.use(mixins); // 全局挂载动态加载的混入
  new Vue({
    el: '#app',
    router,
    components: { App },
    template: '<App/>'
  })
  ```

## 样式

* 如果产品组的UI比较多变，或者逢年过节需要出一个新皮肤来优化用户体验。此时就需要一份样式常量来降低样式的成本。而如果样式代码没有单独维护在一个文件夹下，非常正常合理的存在于``<style>``标签内，那么样式常量就需要在各个标签下都进行引入才能统一使用。这就有了全局样式文件挂载的需求。需要样式常量挂载在所有样式之前。

* ``Vue``实例提供的api并没有提供样式挂载的方式，所以样式的全局挂载本身是基于``sass-loader``与``less-loader``等``webpack``插件配置实现的。在``@vue/cli``脚手架中，已经为开发人员封装好了样式预处理器的配置，样式文件、样式常量引入的语句直接配置即可。

  ```js
  // vue.config.js
  module.exports = {
    css: {
      loaderOptions: {
        // sass-loader, sass与scss通用, 区别在于scss语法需要分号
        sass: {
          addtionalData: `@import "~@/variables.sass"`
        },
        scss: {
          addtionalData: `@import "~@/variables.scss";`
        },
        // less-loader
        less: {
          // `primary` 即为全局变量名
          globalVars: {
            primary: '#fff'
          }
        }
      }
    }
  }
  ```
