# JavaScript - Promise

> ES6引入的一种异步编程方案，它会在未来的某个时间点交付异步操作的结果，这个结果既可以是成功的，也可以是失败的。

## 1. Promise对象

* 一个**Promise**对象代表一次异步操作，且在生成后被立即执行

* **Promise**对象有三种状态：
    
    * **pending** : 初始状态，既不是成功，也不是失败状态
    
    * **fulfilled** : 意味着操作成功完成
    
    * **rejected** : 意味着操作失败

* 通过**new Promise()**创建一个**Promise**对象
    
    * **resolve**将把Promise对象从**pending**变为**fulfilled**状态

    * **reject**将把Promise对象从**pending**变为**rejected**状态

    ```javascript
    const p1 = new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(1);
        }, 100)
    })
    ```

## 2. Promise实例方法

* **then()** 接受两个方法作为参数，分别用来指定成功状态与失败状态的回调，两个方法接受的参数由Promise对象中的**resolve**与**reject**提供。

    ```javascript
    p1.then(val => {
        console.log(val)
    }, err => {
        console.log(err)
    })
    ```

* **catch()** 处理异常的方法，参数为一个方法，与**then**的第二个回调方法类似，可以用于处理失败状态。不同点在于**catch**也可以获取到回调方法中的代码异常，所以实际使用时，经常使用以下写法。

    ```javascript
    p1.then(val => {
        console.log(val)
    }).catch(err => {
        console.log(err)
    })
    ```

* **finally()** 在promise结束时，无论结果是fulfilled或者是rejected，都会执行指定的回调函数。该方法没有参数。

    ```javascript
    p1.then(val => {
        console.log(val)
    }).catch(err => {
        console.log(err)
    }).finally() {
        console.log('end')
    }
    ```

## 3. Promise静态方法

* **Promise.resolve()** 创建一个立即**fulfilled**的**Promise**对象

    ```javascript
    Promise.resolve(1)
    /* 等价于 */
    new Promise(resolve => {
        resolve(1)
    })
    ```

* **Promise.reject()** 创建一个立即**rejected**的**Promise**对象

    ```javascript
    Promise.reject(1)
    /* 等价于 */
    new Promise((resolve, reject) => {
        reject(1)
    })
    ```

* **Promise.all()** 参数为一个**Promise**数组，用于统一处理一组**Promise**对象的异步调用结果

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

* **Promise.race()** 参数也为一个**Promise**对象数组，数组中有一个对象**resolve**时，新对象状态变为**fulfilled**；当有一个对象**reject**时，新对象状态变为**rejected**。**race**方法常用于超时处理。

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

    Promise.race([p1, p2]).then(val => {
        console.log(val)
    }).catch(err => {
        console.log(err)
    });
    /**
     * 由于p2的执行时间更短，所以race结果为fulfilled状态。
     * output:
     * 2
     */
    ```

## 4. 链式调用

在2中，**then()**，**catch()**，**finally()**方法以链状的形式被先后调用，其能够执行的原因在于这三个方法本身都将return一个**Promise**实例，返回实例能够继续调用**Promise**的实例方法。

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