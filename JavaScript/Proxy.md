# Proxy

## setter/getter

JavaScript 支持 setter 和 getter 已经很长时间了。他们用带有 set 和 get 关键字的简单语法来拦截对象的属性访问和值的修改操作。

例如

```javascript
const obj = {
  propValue: 1,
  get prop() {
    console.log("Retrieving property prop");
    return this.propValue;
  },
  set prop(value) {
    console.log("Setting property prop to", value);
    this.propValue = value;
  },
};
```

setter/getter 有多个缺点：它们仅限 get 和 get 操作（显然）。它们不能与相同键（即"常规"属性）的数据入口一起使用。它们不是动态的，必须在对象声明期间用静态的 Object.defineProperty\(\) 方法或通过使用计算值（仅适用于新的浏览器）显式地应用于每个属性。

## Proxy

代理是内置的 JS 对象，可用于拦截和更改与对象相关的不同操作的行为。

```javascript
const originalObj = {
  prop: 1,
  anotherProp: "value",
};
const proxyObj = new Proxy(originalObj, {
  get(obj, prop) {
    console.log("Retrieving property", prop);
    return obj[prop];
  },
  set(obj, prop, value) {
    console.log("Setting property", prop, "to", value);
    obj[prop] = value;
    return true;
  },
});
originalObj.prop;
originalObj.prop = 2;
proxyObj.prop;
```

Proxy 会创建一个新对象供你与之交互，而不是与原始对象进行交互，原始对象在使用 setter/getter 时会直接修改

在使用 Proxy 的情况下，原始对象（也称为 target）用作一种存储。你对其执行的任何操作都会直接影响代理，但不会触发其任何 trap。代理的 [trap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 是执行特定操作时调用的简单方法。它们都是在单个 handler 对象上定义的，然后传递给 Proxy 构造函数。除此之外，它们不仅限于 set\(\) 和 get\(\)

使用静态的 Proxy.revocable\(\) 方法可撤销代理

