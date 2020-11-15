# Vue组件通信整理

## 1. v-bind 与 props

> Vue中使用最为普遍的父子组件通信方式，传递方向，父组件 -&gt; 子组件

父组件通过v-bind绑定参数传递至子组件，子组件通过props同名参数进行接收

```markup
  <!-- 父组件调用子组件 -->
  <div id="parent">
    <child-component :oneProp="oneProp"></child-component>
  </div>
```

```javascript
  /* 子组件接收 */
  var vm = new Vue({
    el: "#children",
    props: {
      oneProp: {
        type: String,
        default: ''
      }
    }
  })
```

子组件在`created`即可接收到来自父组件的props。接收到数据后，可以与data一样，绑定至需要渲染的view层组件上，亦或是监听、修改。

props一个非常显著的特点就是**单项绑定**。在上述样例中，父组件的`oneProp`通过v-bind绑定了一个动态参数，该值是可以改变的。当父组件的`oneProp`发生变化时，即会对子组件对应的props产生影响，并且在有watch监听的情况下，触发监听方法。而当子组件内部对值进行修改时，并不会影响父组件中绑定的参数。

props的单项绑定其实很容易造成父子组件数据不同步的问题，而有些用于通信的props又要保证父子组件所绑定参数的双向性，而这就涉及到子组件向父组件的通信方法。

## 2. vm.$emit 与 v-on

> Vue中又一常用通信方式，传递方向，子组件 -&gt; 父组件

在子组件中使用`vm.$emit`方法，将子组件中的值传递给父组件，父组件则通过`v-on`事件进行接收，与props配合使用即可达成父子组件数据的双向通信。扩充一下上面的代码：

* 父组件部分

  ```markup
  <div id="parent">
    <child-component
      :oneProp="oneProp"
      @on-receive="handleReceive"
    />
  </div>
  ```

  ```javascript
  var parent = new Vue({
    el: "#parent",
    data: {
      oneProp: 1
    }
    methods: {
      handleReceive(arg) {
        this.oneProp = agr;
      }
    }
  })
  ```

* 子组件部分

  ```javascript
  var child = new Vue({
    el: "#children",
    props: {
      oneProp: Number
    },
    mounted() {
      this.oneProp = this.oneProp + 1;
      this.$emit('on-receive', this.oneProp); // 第一个入参用于对应v-on时间名称，第二个入参用于传递参数
    }
  })
  ```

* Vue双向绑定的精髓指令v-model，本质上就是一种Vue已经内置的props，当具有v-model绑定的内容时，子组件可以通过名为`value`的props对绑定内容进行监听，同时可以通过`$emit('input')`将内部值更新后的内容返回给父组件，这里的`input`事件同样是Vue内置的专门用于`v-model`通信的事件。这两格内置的参数与方法对于具有`v-model`属性的自定义组件非常重要。同时，通常在设计时，会在子组件内部定义一个内部参数与内部监听方法，来与父组件传入的`value`进行区分。
* 子组件部分

  ```javascript
  var child = new Vue({
    el: "#children",
    data: {
      innerValue: 0,
    }
    props: {
      value: Number
    },
    watch: {
      value(newVal) { // 外部监听，用于父->子通信传值
        this.innerValue = newVal;
      },
      innerValue(newVal) { // 内部监听，用于子->父通信传值
        this.$emit('input', newVal);
      }
    }
  })
  ```

* 父组件部分

  ```markup
  <!-- 父组件无需定义@input事件，自动接收并改变param内容 -->
  <div id="parent">
    <child-component v-model="param" />
  </div>
  ```

## 3. vm.$ref 与 vm.$parent

`$ref`与`$parent`严格来说并不属于组件数据通信范畴。它们可以但并不推荐使用于数据通信，`$ref`与`$parent`本质上是DOM实例创建成功后，Vue在各个组件中记录的一个组件实例属性，可以在父组件引用子组件时为子组件标记一个ref，父组件就可以从实际上获取到子组件的所有内容。

* 父组件部分

  ```markup
  <div id="parent">
    <child-component ref="child" />
  </div>
  ```

  ```javascript
  var parent = new Vue({
    el: "#parent"
    mounted() {
      console.log(this.$refs.child.innerValue); // 1 mounted时，实例创建完成
    },
    created() {
      console.log(this.$refs); // undefined  实例未创建完成
    }
  })
  ```

* 子组件部分

  ```javascript
  var children = new Vue({
    el: "children"
    data: {
      innerValue: 1
    }
  })
  ```

  可以获取到所有内容，自然也不仅限于参数，包括子组件的方法也同样可以调用。往往通过调用子组件的方法，来改变子组件内部的一些参数。这里其实可以看出Vue组件之间的参数、方法互相调用的自由度其实是很高的，尤其是外部组件调用内部组件的方法来操作内部组件的参数，其实很容易造成数据流的混乱或者不安全性。所以对于方法命名时，可以尽可能的将私有方法与公共方法进行命名或者注释上的区分，来保证安全调用与数据通信。

  而`$parent`则无需定义类似ref的参数，子组件内自然可以获取到父组件的实例，调用方法与`$ref`类似，这里就不过多赘述。

