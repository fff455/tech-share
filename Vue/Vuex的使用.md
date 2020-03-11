# VueX的使用

> **Vuex** 是一个专为 Vue.js 应用程序开发的状态管理模式。其思想借鉴于**Flux**， **Redux**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

* 介绍： [VueX是什么](https://vuex.vuejs.org/zh/)

## 为什么要使用Vuex

* 所谓的状态管理模式的状态，可以理解为一组通过 **Vuex** 进行维护的全局变量。一些用户的个人参数，亦或是多组件共同维护的prop。为了正确维护这些参数，可以使用数据库进行存储，也可以在各类组件之间多次传递。虽然方法可行，但在维护的过程中，可以明显感受非常麻烦。此时使用 **Vuex** 来进行存储与维护，很容易就能化繁为简。当然，**Vuex** 并不适合用于比较小型的应用。

## 引入Vuex

* 安装

  ```shell
  npm install --save vuex
  ```

* vue-cli项目引入vuex

  ```javascript
  import Vue from 'vue';
  import Vuex from 'vuex'; //引入Vuex
  import App from './App'
  import router from './router'

  //使用Vuex
  Vue.use(Vuex);

  //定义Vuex实例
  const store = new Vuex.Store({

  })

  new Vue({
    el: '#app',
    store, //全局挂载Vuex
    router,
    components: { App },
    template: '<App/>'
  })
  ```

## Vuex核心概念

* **state** 用于共享数据存储

* **getter** 用于对共享数据进行处理操作

* **mutation** 用于注册改变数据状态

* **action** 解决异步改变共享数据

* **module** 数据维护模块化

从项目的角度考虑，可以将这五大核心分别进行维护，由于 **module** 相对于其他特征有所区别，所以先不进行引入。

![在项目中使用vuex](./image/vuex.png)


### **State**

> 数据存储的位置，对象类型直接维护数据

* 定义

  ```javascript
  const state = {
    msg: 'Hello Vuex!';
  }
  export default state;
  ```

* 使用
  
  由于store已在Vue实例上进行全局挂载，所以我们可以直接获取到state中所储存的数据，其中this指向Vue实例。可以配合computed计算属性使用，或者在加载组件时进行赋值。

  ```javascript
  this.msg = this.$store.state.msg; // "Hello Vuex!"
  ```

  如果有安装vue-devtool，可以在其中Vuex的Tab页直接看到state所储存的数据。

* mapState函数

  对于state内数据进行实时的监听和使用，很容易想到计算属性computed，比如

  ```javascript
  export default {
    computed: {
      msg1() {
        return this.$store.state.msg1;
      },
      msg2() {
        return this.$store.state.msg2;
      }
    }
  }
  ```

  这样通过计算属性获取与msg同名的state.msg属性，就显得有些冗余了，在参数较少的情况下可能没什么感觉，但在参数多的情况下就会显得非常麻烦，所以我们可以使用mapState函数简化这个过程

  ```javascript
  import { mapState } from 'vuex';
  export default {
    computed: {
      ...mapState({
        msg1: state => state.msg1,
        msg2: state => state.msg2 
      })
    }
  }
  ```


### **getter** 

> 对state中的数据做统一的处理操作，但并不改变数据本身

* 如果没有getter，我们要对state的数据进行一些譬如筛选的操作，可以直接对数据进行

  ```javascript
  export default {
    computed: {
      frontStr() {
        return this.$store.state.substring(0, 5); // "Hello"
      }
    }
  }
  ```

* 由于state中的参数本身具有全局变量的特征，所以常常不止一处会对其进行一些操作，如果这些操作方法类似或者相同，那代码就会显得有些冗余，这个时候就可以使用getter来维护这些方法

* 定义

  ```javascript
  const getters = {
    frontStr: state => state.msg.substring(0, 5);
  }
  export default getters;
  ```

* 使用

  ```javascript
  this.msg = this.$store.getters.frontStr;
  ```

  getter更类似于Vue中的计算属性computed，是state经过某种处理后的储存数据。它所返回的结果会被缓存起来，直到它所依赖的状态数据被改变后才会重新调用进行计算，也因而getter并不能改变state的数据本身。

* mapGetter函数

  与mapState一样，mapGetter也是出于对于代码简化的角度所考虑使用的。由于getter本身不对state数据做修改，其处理效果所返回的结果也与state一样会在计算属性中被使用，所以mapGetter的使用效果与mapState极为类似

  ```javascript
  import { mapGetter } from 'vuex';
  export default {
    computed: {
      ...mapGetter([
        'frontStr',
      ])
    }
  }
  ```

### **mutation**

> 通过在mutation内注册方法，可以对state内的数据进行修改

* 数据修改处理就与字面意思一样容易理解了，我们可以在mutation内定义修改方法来操作state的数据。

* 定义

  ```javascript
  const mutations = {
    // payload为有效荷载数据，用于储存mutation方法入参
    addStr: (state, payload) => {
      state.msg += payload.str1 + payload.str2
    },
  }
  export default mutations;
  ```

* 使用

  ```javascript
  export default {
    methods: {
      addStr() {
        this.$store.commit({
          type: 'addStr',
          str1: 'a',
          str2: 'b',
        })
      }
    },
  }
  ```

* mapMutations函数

  mutations中同样有简化代码的方法mapMutations，与mapGetters不同之处在于，mutations本身都是修改state数据的方法，所以mapMutations多为在methods属性内使用

  ```html
  <template>
    <div>
      <button @click="addStr({str1:'a',str2:'b'})">add</button>
    </div>
  </template>
  ```

  ```javascript
  import { mapMutations } from 'vuex';
  export default {
    methods: {
      ...mapMutations([
        'addStr',
      ])
    }
  }
  ```

* mutation有一个特别要注意的点是，只能使用同步方法，不可异步操作数据，任何异步操作都不会在mutations内生效。那么为了解决异步操作的问题，action就应运而生了。

### **action**

> 功能与mutation类似，但用于储存异步方法

* 在mutation中，我们也提到了异步方法需要action来进行处理，而同步方法已在mutation中处理，所以一般我们可以在action中编写异步逻辑，然后调用mutation中的同步方法，通过混用来达到action需要达到的效果。



### **module**