# Object.defineProperty()简介

> **Object.defineProperty()** 是一个定义在Object构造函数上的方法。从命名就可以看出，此方法用于定义对象的属性。Vue中就是使用此方法完成数据劫持，并实现双向绑定的。通过对此方法的学习，也为之后熟悉Vue双向绑定做准备。

## 1. 定义

* 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。

## 2. 语法

```javascript
Object.defineProperty(obj, prop, descriptor)
```

* **obj** : 要在其上定义属性的对象

* **prop** : 要定义或修改的属性的名称

* **descriptor** : 将被定义或修改的**属性描述符**

* **返回值** : 被传递给函数的对象

## 3. 属性描述符

* 对象里目前存在的属性描述符有两种主要形式：**数据描述符**和**存取描述符**。 

* **数据描述符**是一个具有值的属性，该值可能是可写的，也可能不是可写的。

* **存取描述符**是由getter-setter函数对描述的属性。

* 属性描述符只能同时为两个形式中的一个。

## 4. 属性描述符的键值

把属性本身想象为一个对象，键值参数其实就用于描述该对象的属性。键值共有六个，具有以下表格中的特性。

| 属性名 | **value** | **get**  | **set**  | **writable**  | **enumerable**  | **configurable**  |
| ------- | ------- | ------- | ------- | ------- | ------- | ------- |
| 默认值 | undefined | undefined | undefined | false | false | false |
| 数据描述符 | Yes | No | No | Yes | Yes | Yes |
| 存取描述符 | No | Yes | Yes | No | Yes | Yes |

### 数据描述符独有的两个键值(value, writable)

* **value** : 用于定义属性的值。

* **writable** : 用于描述属性是否可写。

一个很简单的应用 :

```javascript
let coder = {};
Object.defineProperty(coder, 'lang', {
    value: 'JavaScript',
    writable: true
})
console.log(coder.lang); // "JavaScript"

// writable为true时，对象的属性可写
coder.lang = 'Java';
console.log(coder.lang); // "Java"
```

当然，当我们不添加writable属性时，属性的值将不可修改(默认为false)

```javascript
let coder = {};
Object.defineProperty(coder, 'lang', {
    value: 'JavaScript',
})
console.log(coder.lang); // "JavaScript"

// writable为默认值时，对象的属性不可写
coder.lang = 'Java';
console.log(coder.lang); // "JavaScript"
```

### 存取描述符独有的两个键值(get, set)

* **get** : 一个给属性提供获取值的方法，该方法返回值被用作属性值。无默认值则返回undefined。

* **set** : 一个给属性提供设置值的方法。该方法将接受唯一参数，并将该参数的新值分配给该属性。

```javascript
let coder = {};
let tmp = 'JavaScript'
Object.defineProperty(coder, 'lang', {
    value: 'JavaScript',
    writable: true,
    get: () => {
        return tmp;
    },
    set: (val) => {
        tmp = val;
    }
})
// 直接获取对象属性值
console.log(coder.lang); // "JavaScript"

// 设置对象属性值后再次获取
coder.lang = "Java";
console.log(coder.lang); // "Java"
```

### 混合使用两种描述符的键值

当上述两类特有的键值被一起使用时，就违反了描述符本身的定义，出现语法错误。

![数据描述符与存取描述符共存](./image/togetherError.png)

### 数据描述符和存取描述符共同具有的键值(configurable, enumerable)

* **configrable** : 描述属性是否配置，以及可否删除。

* **enumerable** : 描述属性是否会出现在for in 或者 Object.keys()的遍历中。

先对**configurable**进行分析

```javascript
let coder = {};
Object.defineProperty(coder, 'lang', {
    value: 'JavaScript',
    writable: true,
    configurable: false,
    enumerable: true,
})
// 首先，configurable为false时，属性不可删除
delete coder.lang
console.log(coder.lang); // "JavaScript"

// 但如果writable为true可以通过赋值修改内容
coder.lang = "Java";
console.log(coder.lang); // "Java"

// 通过配置方法直接改值则会报错
Object.defineProperty(coder, 'lang', {
    value: 'JavaScript',
})
// Uncaught TypeError: Cannot redefine property: lang
```

有个神奇的特殊情况是，configurable为false时，writable的值可以从true改为false，反之则不行。

再分析**enumerable**，可以发现与其定义完全符合。

```javascript
let coder = {};
coder.lang = "JavaScript";
Object.defineProperty(coder, 'frame', {
    value: 'Vue',
    enumerable: false
})
Object.defineProperty(coder, 'work', {
    value: '996',
    enumerable: true
})

// 通过for in与Object.keys()方法遍历对象
for(let key in coder) {
    console.log(key);
}
// "lang"  "work"
console.log(Object.keys(coder)) // ["lang", "work"]
```

在以上的使用中，我们又可以发现一个特殊的用法，即直接给对象属性赋值。

* 当调用Object.defineProperty()配置属性时，没有赋值的数据描述符取默认值。

* 当直接对属性赋值时，如以上代码块第二行，则**value**以外的数据描述符均为true。