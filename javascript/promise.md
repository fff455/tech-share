# JavaScript - Promise

> ES6 引入的一种异步编程方案，它会在未来的某个时间点交付异步操作的结果，这个结果既可以是成功的，也可以是失败的。

## 1. Promise 对象

* 一个**Promise**对象代表一次异步操作，且在生成后被立即执行
* **Promise**对象有三种状态：
  * **pending** : 初始状态，既不是成功，也不是失败状态
  * **fulfilled** : 意味着操作成功完成
  * **rejected** : 意味着操作失败
* 通过**new Promise\(\)**创建一个**Promise**对象

  * **resolve**将把 Promise 对象从**pending**变为**fulfilled**状态
  * **reject**将把 Promise 对象从**pending**变为**rejected**状态

  ```javascript
  const p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(1);
    }, 100);
  });
  ```

## 2. Promise 实例方法

* **then\(\)** 接受两个方法作为参数，分别用来指定成功状态与失败状态的回调，两个方法接受的参数由 Promise 对象中的**resolve**与**reject**提供。

  ```javascript
  p1.then(
    (val) => {
      console.log(val);
    },
    (err) => {
      console.log(err);
    }
  );
  ```

* **catch\(\)** 处理异常的方法，参数为一个方法，与**then**的第二个回调方法类似，可以用于处理失败状态。不同点在于**catch**也可以获取到回调方法中的代码异常，所以实际使用时，经常使用以下写法。

  ```javascript
  p1.then((val) => {
    console.log(val);
  }).catch((err) => {
    console.log(err);
  });
  ```

* **finally\(\)** 在 promise 结束时，无论结果是 fulfilled 或者是 rejected，都会执行指定的回调函数。该方法没有参数。

  ```javascript
  p1.then(val => {
      console.log(val)
  }).catch(err => {
      console.log(err)
  }).finally() {
      console.log('end')
  }
  ```

## 3. Promise 静态方法

* **Promise.resolve\(\)** 创建一个立即**fulfilled**的**Promise**对象

  ```javascript
  Promise.resolve(1);
  /* 等价于 */
  new Promise((resolve) => {
    resolve(1);
  });
  ```

* **Promise.reject\(\)** 创建一个立即**rejected**的**Promise**对象

  ```javascript
  Promise.reject(1);
  /* 等价于 */
  new Promise((resolve, reject) => {
    reject(1);
  });
  ```

* **Promise.all\(\)** 参数为一个**Promise**数组，用于统一处理一组**Promise**对象的异步调用结果

  当数组中所有的对象都为**resolve**时，新对象状态变为**fulfilled**

  ```javascript
  Promise.all([Promise.resolve(1), Promise.resolve(2)]).then(val => {
      console.log(val)
  }).catch(err => {
      console.log(err)
  })
  /**
   * output:
   * [1, 2]
   * /
  ```

  当数组中有一个对象**reject**时，新对象变为**rejected**

  ```javascript
  Promise.all([Promise.resolve(1), Promise.reject(2), Promise.reject(3)])
      .then(val => {
          console.log(val)
      }).catch(err => {
          console.log(err)
      })
  /**
   * output:
   * 2
   * /
  ```

* **Promise.race\(\)** 参数也为一个**Promise**对象数组，数组中有一个对象**resolve**时，新对象状态变为**fulfilled**；当有一个对象**reject**时，新对象状态变为**rejected**。**race**方法常用于超时处理。

  ```javascript
  const p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(1);
    }, 500);
  });

  const p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(2);
    }, 100);
  });

  Promise.race([p1, p2])
    .then((val) => {
      console.log(val);
    })
    .catch((err) => {
      console.log(err);
    });
  /**
   * 由于p2的执行时间更短，所以race结果为fulfilled状态。
   * output:
   * 2
   */
  ```

## 4. 链式调用

在 2 中，**then\(\)**，**catch\(\)**，**finally\(\)** 方法以链状的形式被先后调用，其能够执行的原因在于这三个方法本身都将 return 一个**Promise**实例，返回实例能够继续调用**Promise**的实例方法。

```javascript
new Promise((resolve, reject) => {
    resolve(1);
}).then(val => {
    console.log(val)
    return new Promise((resolve, reject) => {
        resolve(2);
    })
}).then(val => {
    console.log(val)
    return new Promise((resolve, reject) => {
        resolve(3);
    })
}).then(val => {
    console.log(val);
})
/**
 * output:
 * 1
 * 2
 * 3
 * /
```

## 5. Promise、setTimeout、async/await 执行顺序问题

### a. js 事件循环机制\(Event Loop\)

* 在了解这三者的执行问题前，先来回顾一下 js 事件循环机制。众所周知，js 自古以来就是一门单线程，非阻塞的语言。单线程意味着只有一个主线程来执行所有的任务，也就是说同一时间只能执行一个任务。
* **Event Loop**：分为主线程、宏任务、微任务三部分
  * 主线程：可以直接执行的 js 代码
  * 宏任务：**setTimeout**，**setInterval**
  * 微任务：**Promise.then\(\)**，**new Promise**主体部分的内容属于宏任务，可以直接执行。另外带 **async** 关键字的方法本身会返回一个 **Promise** 对象，可以理解为有 **await** 关键字的方法及其之前的部分为 **Promise** 对象的主体内容。带关键字**await**的方法之后的内容就是 **Promise.then\(\)** 部分。
* 执行机制关键点：
  * 优先执行主线程
  * 遇到宏任务时，将其放入宏队列，遇到微任务时，将其放入微队列
  * 主线程执行完毕后，执行微队列中的所有微任务直至清空
  * 执行一个宏任务，宏任务完成后，查看主线程或微队列是否为空，若为空则继续执行
  * 对于 setTimeout 而言，会在时间到了之后才会进入宏队列，而进入宏队列不意味着执行，也因此 setTimeout 内的代码的时机执行间隔是随机且略大于设定时间的。
* 了解了机制之后，举一个例子

  ```javascript
  1  async function async1(){
  2      console.log('async1 start')
  3      await async2()
  4      console.log('async1 end')
  5  }
  6  async function async2(){
  7      console.log('async2')
  8  }
  9  console.log('script start')
  10 setTimeout(function(){
  11     new Promise(function(resolve){
  12         console.log('promise3')
  13         resolve()
  15     }).then(function(){
  16         setTimeout(function(){
  17             console.log('setTimeout3')
  18         }, 0)
  19         console.log('promise4')
  20     })
  21     console.log('setTimeout1')
  22 },0)
  23 setTimeout(function(){
  24     console.log('setTimeout2')
  25 }, 0)
  26 async1();
  27 new Promise(function(resolve){
  28     console.log('promise1')
  29     resolve();
  30 }).then(function(){
  31     console.log('promise2')
  32 })
  33 console.log('script end')
  ```

* 挨个进行分析
  1. 首先主线程从 9 开始执行，毫无疑问先输出 script start；
  2. 10~22，23~25。两个 setTimeout 方法，宏任务，由于主线程还没执行结束，所以先放入宏队列，分别记为 M1 和 M2；
  3. 26 为带 async 关键字的方法，先进入，2~3 为主线程代码，所以输出 async1 start，然后执行 async2\(\);
  4. async2\(\)这个方法的 async 关键字在这个例子中其实是多余的，该方法可以视作一个普通的方法，所以为主线程内容，输出 async2;
  5. 4~5 为 await 之后，相当于 Promise.then 部分，主线程还没结束，扔进微队列，记为 m1；
  6. 主线程现在到了 27，28 为 Promise 对象主体，主线程内容，输出 promise1；
  7. 30~32，Promise.then 部分，扔进微队列，记为 m2；
  8. 33，主线程，输出 script end，到此主线程暂时执行完毕；
  9. 主线程完毕，开始看微队列，先进先出原则，首先执行 m1，输出 async1 end；
  10. 微队列没有结束，继续执行微队列中的 m2，故输出 promise2，到此微队列暂时也告一段落；
  11. 接下来开始执行宏队列中的 M1，11~12 相当于主线程代码块，直接运行，输出 promise3；
  12. 13 为 resolve 方法，执行后将 15~20 扔进微队列，记为 m3；
  13. 21 为当前宏任务主代码块内容，直接输出 setTimeout1。至此，M1 宏任务结束;
  14. 此时，微队列增加 m3，故要先去执行微队列，发现 16~18 又是一个宏任务，记为 M3；
  15. 直接执行 19，输出 promise4；
  16. 微队列再次清空，执行宏队列中的 M2，输出 setTimeout2；
  17. 一个宏任务结束，查看主线程与微队列都为空，继续执行宏队列的 M3，输出 setTimeout3，至此，代码全部执行完成。
* 执行结果

  ```javascript
  script start
  async1 start
  async2
  promise1
  script end
  async1 end
  promise2
  promise3
  setTimeout1
  promise4
  setTimeout2
  setTimeout3
  ```

* 结语： 代码执行顺序问题引发的 bug 在平时工作中经常出现，了解 js 代码以及各个异步编程方法的执行机制，可以有效规避这类问题。

