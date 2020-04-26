# Vue响应式原理及总结

> 双向绑定是Vue框架的最显著特点之一，它通过Object.defineProperty()劫持数据来实现非侵入式的数据响应。以此监听数据变化并实现DOM变化。本文将通过阅读Vue官方文档，并结合开发经验，对Vue响应式原理进行简要介绍与总结

参考链接：

[Vue响应式原理官方文档](https://cn.vuejs.org/v2/guide/reactivity.html)

[Object.defineProperty()简介](https://xiaobaihaha0001.gitbook.io/fe-share/javascript/object.defineproperty-jian-jie)

## 如何追踪变化

* 既然要谈数据响应与双向绑定，那自然离不开Vue实例中的data，data返回的内容本身就是一个大的Object。当然内部数据的数据结构自然也会有对象。

* 在Vue实例初始化时，会对data中的所有数据进行一次遍历，通过Object.defineProperty方法，将数据转化为用户不可见的getter/setter。

* 这个转化的过程相当于为Vue实例的数据建立了一个监听的钩子，在需要获取data值，或者设置data值时，都会触发钩子进行响应。

* 从Vue的实际操作来讲，它会为每一个组件实例都对应设立一个watcher实例，watcher与data之间的联系就是上一点中所说的钩子。

  * 在控件dom进行初次渲染的时候，由于数据本身在Vue实例初始化时，已经遍历过一次，所以能够通过数据的getter获取。

  * 如果人为改变data，会触发数据的setter，通知(notify)watcher再触发对应的dom的相应变化。

  ![Vue响应式原理](./image/data.png)

## 监测变化的注意事项

* 平时在开发中经常会遇到这样的情况，对于一个表单(form)而言，我们需要绑定一个Object。但我们必须将表单的属性在data中写明。

  ```javascript
  /* correct */
  data() {
    return {
      formItem: {
        inputA: '',
        inputB: '',
      }
    }
  }
  /* incorrect */
  data() {
    return {
      formItem: {}
    }
  }
  ```

* 这就引申出了一个问题，控件的Object数据与watcher的钩子在初始化时已经建立，后续再人为增加或删除属性，都不会触发watcher。

* 与此同时，如果后续在根级别增加响应式数据，也同样无法触发，同样是源于Vue实例在初始化时进行了data遍历，而后续不会再进行遍历。注：根级别就是data内部第一层属性的数据，相当于上面代码中的formItem一级。

* 对于根级数据来说，Vue是要求我们进行完备的定义的。而对于非根级的对象属性数据来说，可通过vm.$set()方法增加对象属性，并强行触发监听事件。

  ```javascript
  this.$set(this.someObject,'oneProperty', value)
  ```

* 上述方法是对于对象的单个属性而言的，官网中提供了用Object.assign()来处理多个属性修改的写法，但在个人实际应用中运用较少。实际应用中常有与data的Object浅拷贝的数据，若直接移走绑定数据对象的指针，容易产生更多的问题。

  ```javascript
  this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
  ```

## 异步更新队列

* Vue的异步更新队列机制在[nextTick()的使用](https://xiaobaihaha0001.gitbook.io/fe-share/vue/nexttick-de-shi-yong)这一节中进行了介绍，这里不再赘述。

## 结语

Vue响应式原理相对来说还是挺容易理解的。在理解之后对于开发工作来说，可以少踩很多基础的小坑。但当然在Vue的响应式数据中，并不只有data而已，对于watch、computed、props属性中数据的机制，会更复杂一些，将在后续进行学习讨论。