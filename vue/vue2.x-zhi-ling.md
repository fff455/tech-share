# Vue2.x指令

* 指令\(Directives\)，指带有`v-`前缀的特殊属性。
* 当表达式的值改变时，将其产生的连带影响，响应式地作用于DOM，是Vue双向绑定最基础的表现。

## 内置指令

* **v-text**

  ```markup
  <div v-text="text"></div>
  ```

  绑定的参数与将作为元素内部的`textContent`，等价于

  ```markup
  <div>{{text}}</div>
  ```

* **v-html**

  ```markup
  <div v-html="html"></div>
  ```

  绑定的参数将作为元素的`innerHTML`内容，要注意防止xss攻击

* **v-bind**

  ```markup
  <div v-bind:class="className"></div>
  ```

  Vue中最基本的参数双向绑定，当用于原生的元素标签时，可以将参数绑定至标签的原生属性上，当用于封装的组件时，将作为`props`传入组件内。通常会使用以下的写法

  ```markup
  <div :class="className"></div>
  ```

* **v-on**

  ```markup
  <button v-on:click="handleClick(args, $event)">click me!</button>
  ```

  用于绑定事件，事件方法定义在`methods`内。对于有原生事件的标签来说，`v-on`能够直接支持。而对于封装的组件而言，可以在组件内使用`$emit()`来定义事件触发的条件。入参参数在没必要的情况下能够省略。通常会使用以下写法

  ```markup
  <button @click="handleClick(args, $event)"></button>
  ```

  * 事件修饰符：

  ```markup
  <!-- 使用方法 -->
  <button v-on:click.stop="handleClick">click me!</button>
  ```

  `.stop`: 阻止冒泡，调用`event.stopPropagation()`

  `.prevent`: 阻止默认事件，调用`event.preventDefault()`，如用于表单提交事件

  `.capture`: 添加事件侦听器时使用事件捕获模式，即元素自身触发的事件先在事件方法内处理，然后才交由内部元素进行处理

  `.self`: 只当事件在该元素本身（比如不是子元素）触发时触发回调

  `.once`: 事件只触发一次

  `.passive`: 用于滚动事件，滚动事件的默认行为 \(即滚动行为\) 将会立即触发

* **v-model**

  ```markup
  <input v-model="inputValue">
  ```

  元素双向绑定的数据，与`v-bind`类似，相比于前者，在组件封装中，使用`v-model`绑定的数据更偏向于组件的核心数据，如表单绑定的数据，输入框内的内容等。

  注意`v-model`前不要加 `:` ，否则其实际意义将会变成`v-bind:v-model`，与一个普通的`props`无异。

  在组件内部进行接收或者监听时，都使用`value`。另外，由于`prop`本身的单向传输性，当组件内部数据更新且需要返回给外部时，得使用`$emit('input', value)`，以下为一个简单的`v-model`在组件内外双向绑定的写法

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

  ```markup
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

  ```markup
  <div>1</div>
  <div>2</div>
  <div>3</div>
  ```

  在Vue2.x中，要注意想要触发dom的变化，就需要触发监听数组变化的钩子，而当数组内对象层级过深时，钩子将不会触发，于是会导致model层更新，而view层没有响应的情况，此时可以使用`$forceUpdate`强制触发钩子。

  `v-for`的key属性，可以理解为一个主键，可用于提升dom重载时的性能。

* **v-if与v-show**

  ```markup
  <div v-if="isDicShow">1</div>
  <span v-show="isSpanShow">2</span>
  ```

  与`v-if`相伴的还有`v-else-if`与`v-else`，将作为`v-if`的一类一起讨论。

  `v-if`和`v-show`都通过赋值其的表达式的真假来控制所在元素的是否显示。两者的区别在于`v-if`绑定的表达式的真假性变化时，会导致元素的渲染或者销毁。而`v-show`仅仅是控制元素的`display`属性是否为`none`，绑定值为false时，我们依然可以通过审查元素找到该元素。关于两者的区别，其实是个老生常谈的问题，本质上，还是一个考虑性能的问题。

  * `v-if`无初始渲染消耗，但有较高的切换消耗，`v-show`则相反。
  * 两者的使用要根据实际业务场景进行考量，对首次打开性能要求较高的界面，可以考虑使用`v-if`，对于需要频繁切换的元素，则可以考虑使用`v-show`

* **v-cloak**

  ```markup
  <div class="demo-class" v-cloak>{{msg}}</div>
  ```

  ```css
  [v-cloak] {
    display: none;
  }
  ```

在开启禁止浏览器缓存或者网络比较慢的时候，即`vue.js` 加载较慢的时候，由于`Vue`实例并没有编译完成，所以对于类似 `{{msg}}` 的 `Mustache` 标签会被直接展示出来。只有当 `Vue` 编译加载完成后，参数的真正内容才会被展示出来。这就导致了界面渲染时会有一小段时间的闪动问题。

而 `v-cloak` 主要就是用于优化这个用户体验的问题，用CSS选择器来操作拥有该指令的样式，设置其为 `display:none` ，那么在`vue.js`加载期间，该元素就不会被展示出来，直到 `Vue` 实例加载完毕。

事实上 `v-cloak` 指令在实际编码过程中，用到的机会还是比较少的。在较为大型、工程化的项目当中，都会使用 `vue-router` 实现路由挂载，所以在非首页加载的情况下，`Vue`实例都早已加载完成，此时就不再有使用该指令的必要。

* **v-pre**

  ```markup
  <div v-pre>{{msg}}</div>
  ```

拥有 `v-pre` 指令的元素与其子元素会直接跳过 `Vue` 的编译过程，当作普通的dom进行渲染。跳过编译过程一来可用于在必要的时候展示 `{{msg}}` 这类 `Mustache` 标签。二来可用于缩短一些不含需要编译内容的dom的加载时间，从而提高整个界面的加载效率。从实际使用情况来说，对性能的提升还是较为有限的，不过可以作为界面性能优化的一个方面加以实施。

* **v-once**

  ```markup
  <div v-once>{{msg}}</div>
  ```

同样是一个用于提高界面性能的指令，可以从字面意思来理解，使用`v-once`指令的元素只会被编译一次，也就是说类似于`{{msg}}`的标签内的参数会被正确展示出来，但有且仅有这一次渲染，若界面出现重新渲染，该元素就不再会进行编译，只会展示第一次编译的内容。该指令一般用于有需要编译的`Vue`语法，但其依旧为静态内容，所以仅需编译一次，可以与`v-pre`相区分。

## 自定义指令

`Vue`除去以上内置指令外，还可以让开发人员根据生产需要开发自定义指令。通过对内置指令的介绍，很容易可以认识到，指令的作用其实主要用于对于DOM元素的操作上。比如官方文档里给的一个例子，就是用于做输入框获取焦点的。

自定义指令与`Vue-components`类似，可以进行全局挂载。比如获取输入框焦点这类比较常见的交互场景，就可以作为一个全局的自定义指令进行挂载。

```javascript
// 注册一个全局自定义指令 `v-focus`
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时……
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  }
})
```

同样的，也可以仅在某个界面内进行局部挂载。

```javascript
new Vue({
  el: "#app",
  directives: {
    focus: {
      inserted: function(el){
        el.focus();
      }
    }
  }
})
```

对于指令功能的实现，与`Vue`实例类似，都具有一个完整的生命周期及其对应的钩子函数，指令的生命周期整体如下。

```text
bind => inserted => update => componentUpdated => unbind
```

对于生命周期的具体解释，以及钩子函数各个入参的含义，[官方文档](https://cn.vuejs.org/v2/guide/custom-directive.html)都已经给出了很明朗的解释，就不赘述。

另外，对于官方文档中提及的动态参数指令 `v-mydirective:[argument]="value"`，即可以在指令上绑定一个`[arguement]`来作为组件实例更新的一个入参，由于入参是一个动态的参数，所以可使指令的定义更加灵活。以下例子中的`dir`就是一个传入的入参，对应于指令方法中的`arg`。

```markup
<div v-pin:[dir]="200"></div>
```

```javascript
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    var s = (binding.arg == 'left' ? 'left' : 'top')
    el.style[s] = binding.value + 'px'
  }
})
```

而其实指令等号后面的部分同样也可以作为一个动态的参数传入，所以当等号后面部分的参数更新时，在`update`钩子内进行更新，同样也可以实现动态参数指令的效果。

