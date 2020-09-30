# JavaScript 原型(1) - 构造函数

> 对于 JS 原型和原型链的理解一直不是很到位，而构造函数又是其中的基础。所以先从构造函数开始，以一种简单易懂的方式来理解构造函数，为之后理解原型与原型链做好准备。

---

## 1. 构造函数的含义

- 构造函数本身是一个函数，当使用 **new** 关键字来进行调用时，这个函数就成为了一个构造函数；

- JS 构造函数的理念与 C++、Java 等面向对象语言相类似，构建了一个对象实例。

```javascript
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
}

let Bai = new PerSon("XiaoBai", "Male");
console.log(Bai); // {name: "Xiaobai", gender: "Male"}
```

---

## 2. 构造函数的意义

当我们要定义一堆同类型对象的时候，最土味的做法就是挨个定义，

```javascript
let White = { name: "XiaoBai", gender: "Male" };
let Black = { name: "XiaoHei", gender: "Female" };
```

很明显这非常麻烦，所以有了构造函数，我们就可以换一种写法，

```javascript
let White = new Person("XiaoBai", "Male");
let Black = new Person("XiaoHei", "Male");
```

还是显得有些麻烦，但当对象参数、对象数量比较多的时候，就会方便许多。当对象用有相同的方法时，使用构造函数封装对象，也能显著减少代码复用。

---

## 3. 构造函数的执行过程

- 使用 **new** 关键字调用构造函数时，创建了一个对象实例，并在内存中创建了一块新的空间，记为 **m**；

- 构造函数内部的 **this** 将会指向该空间 **m**，也就是说每一次调用构造函数都将会创建一个新的对象实例，并创建一块新的空间(记住这个空间创建，与原型对象相区别)；

- 正常执行构造函数内部代码；

- 默认返回构造函数 **this** ，而 **this** 指向 **m**，被赋值的变量也就指向了这块内存空间，同时被标记为一个对象实例;

- 当然也有非返回默认的情况：

  - 当返回一个基础数据类型时，最终返回的依旧是 this

  ```javascript
  function Person() {
    this.name = "XiaoBai";
    return "XiaoHong";
  }

  let Bai = new PerSon();
  console.log(Bai.name); // "Xiaobai"
  ```

  - 而将数组、对象作为返回值时，最终将会返回该数组、对象

  ```javascript
  function Person() {
    this.name = "XiaoBai";
    return { name: "XiaoHong" };
  }

  let Bai = new PerSon();
  console.log(Bai.name); // "XiaoHong"
  ```

github 能直接修改么 试试看
