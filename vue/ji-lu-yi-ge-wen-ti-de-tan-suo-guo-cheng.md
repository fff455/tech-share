# 记录一个问题的探索过程

> 在本周工作中，碰到了一个比较棘手的问题。表格绑定的数组元素更新后，表格无法重新渲染的问题。趁着周末的时间解决一下。

为了提高还原度，也为了简化问题，所以用Vue写了一个demo

模板：

```markup
<template>
    <div>
        <p
            v-for="(item, index) in showObj.rows"
            :key="index"
            :style="`color:${item.color}`">
            Hello World!
        </p>
        <button @click="handleAdd">Add</button>
        <button @click="handleChange">Change</button>
        <button @click="handleClear">Clear</button>
    </div>
</template>
```

Vue实例

```javascript
export default {
    name: 'HomePage',
    data() {
        return {
            showObj: {
                rows: []
            },
            obj: {
                rows: [{
                    color: 'Red'
                }, {
                    color: 'Green'
                }, {
                    color: 'Blue'
                }]
            }
        }
    },
    methods: {
        handleAdd() {
            let red = Math.floor(Math.random()*256);
            let green = Math.floor(Math.random()*256);
            let blue = Math.floor(Math.random()*256);
            this.obj.rows.push({
                color: `rgb(${red},${green},${blue})`
            })
        },
        handleClear() {
            this.obj.rows = this.obj.rows.slice(0, 3);
        },
        handleChange() {
            this.obj.rows.splice(Math.floor(Math.random() * this.obj.rows.length), 1, {
                color: 'white'
            })
        },
    },
    mounted() {
        this.showObj.rows = this.obj.rows;
    }
}
```

* demo中对DOM绑定的数组进行了新增元素与更新元素两种方式的变更，可以发现数据与渲染都正确进行。所以可以排除Vue没有监听到数据变化的问题。
* 在对数组进行初始化操作，只保留前三项的操作后，再次对数组进行Add与Change操作就会发现数据与渲染均不正确。
* 复现问题以后思考造成问题的原因。注意到用于渲染展示的数组是与进行变更的缓存数组通过浅拷贝绑定的。如果在操作过程中缓存的数组指针丢失，那么确实会造成渲染不正确的情况。

```javascript
handleClear() {
  console.log(this.obj.rows === this.showObj.rows)
  this.obj.rows = this.obj.rows.slice(0, 3);
  console.log(this.obj.rows === this.showObj.rows)
}
```

* 于是检查是否指针丢失，可以发现输出结果是false与true。
* 到这里，其实问题的结果已经很明了了，由于slice方法本身并不改变原数组，而在内存中创建了一个新的空间储存新数组，再次赋值后指针就丢失了。
* 回头想想其实是一个很简单的js问题，但在工作中，却因为代码量较多等种种原因，花了很长时间没有解决。希望能在之后的编码中多多注意，谨记。

