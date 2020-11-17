# nextTick\(\)的使用

## vm.nextTick\(\[callback\]\)

* 参数：
  * { **Function** } \[ **callback** \]
* 用法：
  * 将回调延迟到下次 **DOM** 更新循环之后执行。在修改数据之后立即使用它，然后等待 **DOM** 更新。它跟全局方法 **Vue.nextTick\(\)** 一样，不同的是回调的 **this** 自动绑定到调用它的实例上。
* 示例：

  * 模板

    ```markup
    <template>
    <div>
      <p ref="msg">{{message}}</p>
      <button @click="handleMsgChange()">modify</button>
    </div>
    </template>
    ```

  * Vue实例

  ```javascript
  new Vue({
    data: {
      message: 'origin'
    },
    methods: {
      handleMsgChange() {
        // 修改数据
        this.message = 'changed';
        // DOM 还没有更新
        console.log(this.$refs.msg.innerHTML)
        this.$nextTick(() => {
            // DOM 现在更新了
            // `this` 绑定到当前实例
            console.log(this.$refs.msg.innerHTML)
        })
      }
    }
  })
  ```

  * output:

  ```text
  origin
  changed
  ```

* 结果说明：

  > 正如用法中所描述的那样，**nextTick** 的回调方法将在 **DOM** 更新后调用，所以直接在方法内改动数据并不影响 **DOM**，但在nextTick中则发生了变化。同时也说明了 **Vue** 的双向绑定并不是实时的，**Vue** 实例的 **data** 更新后，对应 **DOM** 将会在某一个时刻进行更新。

* 注意点：

  > **Vue** 生命周期中， **created** 在 **DOM** 渲染前执行，所以需要特别注意对于nextTick的合理使用，使 **DOM** 加载正确的数据。

## 异步更新队列

* **Vue** 中数据发生变化时，**DOM** 的更新是异步执行的。只要侦听到数据变化，**Vue** 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个监听钩子被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 **DOM** 操作是非常重要的。然后，在下一个的事件循环 “**tick**” 中，**Vue** 刷新队列并执行实际 \(已去重的\) 工作。

