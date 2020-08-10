# Vue2.x指令

* 指令(Directives)，指带有``v-``前缀的特殊属性。

* 当表达式的值改变时，将其产生的连带影响，响应式地作用于DOM，是Vue双向绑定最基础的表现。

## 内置指令

* **v-text**

  ```html
  <div v-text="text"></div>
  ```

  绑定的参数与将作为元素内部的``textContent``，等价于

  ```html
  <div>{{text}}</div>
  ```

* **v-html**

  ```html
  <div v-html="html"></div>
  ```

  绑定的参数将作为元素的``innerHTML``内容，要注意防止xss攻击

* **v-bind**

  ```html
  <div v-bind:class="className"></div>
  ```

  Vue中最基本的参数双向绑定，当用于原生的元素标签时，可以将参数绑定至标签的原生属性上，当用于封装的组件时，将作为``props``传入组件内。通常会使用以下的写法

  ```html
  <div :class="className"></div>
  ```

* **v-on**

  ```html
  <button v-on:click="handleClick(args, $event)">click me!</button>
  ```

  用于绑定事件，事件方法定义在``methods``内。对于有原生事件的标签来说，``v-on``能够直接支持。而对于封装的组件而言，可以在组件内使用``$emit()``来定义事件触发的条件。入参参数在没必要的情况下能够省略。通常会使用以下写法

  ```html
  <button @click="handleClick(args, $event)"></button>
  ```

  * 事件修饰符：

  ```html
  <!-- 使用方法 -->
  <button v-on:click.stop="handleClick">click me!</button>
  ```

  ``.stop``: 阻止冒泡，调用``event.stopPropagation()``

  ``.prevent``: 阻止默认事件，调用``event.preventDefault()``，如用于表单提交事件

  ``.capture``: 添加事件侦听器时使用事件捕获模式，即元素自身触发的事件先在事件方法内处理，然后才交由内部元素进行处理

  ``.self``: 只当事件在该元素本身（比如不是子元素）触发时触发回调

  ``.once``: 事件只触发一次

  ``.passive``: 用于滚动事件，滚动事件的默认行为 (即滚动行为) 将会立即触发

* **v-model**

  ```html
  <input v-model="inputValue">
  ```

  元素双向绑定的数据，与``v-bind``类似，相比于前者，在组件封装中，使用``v-model``绑定的数据更偏向于组件的核心数据，如表单绑定的数据，输入框内的内容等。

  注意``v-model``前不要加 ``:`` ，否则其实际意义将会变成``v-bind:v-model``，与一个普通的``props``无异。

  在组件内部进行接收或者监听时，都使用``value``。另外，由于``prop``本身的单向传输性，当组件内部数据更新且需要返回给外部时，得使用``$emit('input', value)``，以下为一个简单的``v-model``在组件内外双向绑定的写法
  
  ```javascript
  export default {
    data() {
      return {
        innerValue: '',
      }
    }
    props: {
      value: [String, Number]
    },
    watch: {
      value(newVal) {
        this.innerValue = newVal;
      },
      innerValue(newVal) {
        this.$emit('input', newVal);
      }
    }
  }
  ```

* **v-for**

  ```html
  <div v-for="item in items" :key="item">{{item.content}}</div>
  ```

  ```javascript
  export default {
    data() {
      return {
        items: [1,2,3]
      }
    }
  }
  ```

  操作dom的Vue指令，通过遍历一个数组来达到重复渲染当前元素的效果，如上述代码的实际渲染效果为

  ```html
  <div>1</div>
  <div>2</div>
  <div>3</div>
  ```

  在Vue2.x中，要注意想要触发dom的变化，就需要触发监听数组变化的钩子，而当数组内对象层级过深时，钩子将不会触发，于是会导致model层更新，而view层没有响应的情况，此时可以使用``$forceUpdate``强制触发钩子。

  ``v-for``的key属性，可以理解为一个主键，可用于提升dom重载时的性能。

* **v-if与v-show**

  ```html
  <div v-if="isDicShow">1</div>
  <span v-show="isSpanShow">2</span>
  ```

  与``v-if``相伴的还有``v-else-if``与``v-else``，将作为``v-if``的一类一起讨论。

  ``v-if``和``v-show``都通过赋值其的表达式的真假来控制所在元素的是否显示。两者的区别在于``v-if``绑定的表达式的真假性变化时，会导致元素的渲染或者销毁。而``v-show``仅仅是控制元素的``display``属性是否为``none``，绑定值为false时，我们依然可以通过审查元素找到该元素。关于两者的区别，其实是个老生常谈的问题，本质上，还是一个考虑性能的问题。
  
  * ``v-if``无初始渲染消耗，但有较高的切换消耗，``v-show``则相反。

  * 两者的使用要根据实际业务场景进行考量，对首次打开性能要求较高的界面，可以考虑使用``v-if``，对于需要频繁切换的元素，则可以考虑使用``v-show``

* **v-cloak**

* **v-pre**

* **v-once**

## 自定义指令