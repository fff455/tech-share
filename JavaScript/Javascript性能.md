# Javascript 性能

用控制台是测不出性能的，因为控制台本质上是个套了一大堆安全机制的 eval，它的沙盒化程度很高。同时，不同浏览器的不同版本性能可能就不一样,这里为了统一,node 环境用来测试更好.

## 循环

```javascript
// for 测试(for和while其实差不多,这里我们只测试for循环)
console.time("for");
for (var i = 0; i < arr.length; i++) {
  arr[i];
}
console.timeEnd("for");

// for loop测试
console.time("for");
var sum = 0;
for (var i = 0; i < arr.length; i++) {
  sum += arr[i];
}
console.timeEnd("for");

// for loop缓存测试
console.time("for cache");
var sum = 0;
var len = arr.length;
for (var i = 0; i < len; i++) {
  sum += arr[i];
}
console.timeEnd("for cache");

// for loop倒序测试
console.time("for reverse");
var sum = 0;
var len = arr.length;
for (i = len - 1; i > 0; i--) {
  sum += arr[i];
}
console.timeEnd("for reverse");

//forEach测试
console.time("forEach");
var sum = 0;
arr.forEach(function(ele) {
  sum += ele;
});
console.timeEnd("forEach");

//ES6的for of测试
console.time("for of");
var sum = 0;
for (let i of arr) {
  sum += i;
}
console.timeEnd("for of");

// for in 测试
console.time("for in");
var sum = 0;
for (var i in arr) {
  sum += arr[i];
}
console.timeEnd("for in");
```

最后在 node 环境下各自所花费的不同时间:
|循环类型|耗费时间(ms)|
|-|-|
|for|约 11.998|
|for|cache 约 10.866|
|for 倒序|约 11.230|
|forEach|约 400.245|
|for in|约 2930.118|
|for of|约 320.921|

前三种原始的 for 循坏一个档次,然后 forEach 和 for of 也基本属于一个档次,for of 的执行速度稍微高于 forEach,最后最慢的就是 for in 循环了,差的不是几十倍的关系了

### 原因分析

for in 一般是用在对象属性名的遍历上的，由于每次迭代操作会同时搜索实例本身的属性以及原型链上的属性，所以效率肯定低下.

for...in 实际上效率是最低的。这是因为 for...in 有一些特殊的要求，具体包括：

1. 遍历所有属性，不仅是 ownproperties 也包括原型链上的所有属性。
2. 忽略 enumerable 为 false 的属性。
3. 必须按特定顺序遍历，先遍历所有数字键，然后按照创建属性的顺序遍历剩下的。

遍历数组属性目前有:for-in 循环、Object.keys()和 Object.getOwnPropertyNames(),三种区别：

- for-in 循环:会遍历对象自身的属性,以及原型属性,包括 enumerable 为 false(不可枚举属性);
- Object.keys():可以得到自身可枚举的属性,但得不到原型链上的属性;
- Object.getOwnPropertyNames():可以得到自身所有的属性(包括不可枚举),但得不到原型链上的属性,Symbols 属性
  也得不到.

### 总结

1. 能用 for 缓存的方法循环就用 for 循坏,性能最高,写起来繁杂;
2. 不追求极致性能的情况下,建议使用 forEach 方法,干净，简单，易读，短，没有中间变量，没有成堆的分号，简单非常
   优雅;
3. 想尝鲜使用 ES6 语法的话,不考虑兼容性情况下,推荐使用 for of 方法,这是最简洁、最直接的遍历数组元素的语法,该方
   法避开了 for-in;循环的所有缺陷与 forEach()不同的是，它可以正确响应 break、continue 和 return 语句.
4. 能避免 for in 循环尽量避免,太消费性能,太费时间,数组循环不推荐使用.
   条件语句

## 条件语句

条件语句有 if-else 和 switch-case

if-else 最坏的情况下要做最后一次次判断才能返回正确的结果，一个优化策略是将最可能的取值提前判断，比如 value 最可能等于 5 或者 10，那么将这两条判断提前。但是通常情况下并不知道最可能的选择，这时可以采取二叉树查找策略进行性能优化。

switch-case 语句让代码显得可读性更强，而且 switch-case 语句还有一个好处是如果多个 value 值返回同一个结果，就不用重写 return 那部分的代码。一般来说，当 case 数达到一定数量时，swtich-case 语句的效率是比 if-else 高的，因为 switch-case 采用了[branch table](https://en.wikipedia.org/wiki/Switch_statement)索引来进行优化，当然各浏览器的优化程度也不一样。

汇编上比较[if-else 和 switch-case](http://www.2cto.com/os/201404/291376.html)

### 总结

1. 当只有两个 case 或者 case 的 value 取值是一段连续的数字的时候，我们可以选择 if-else 语句;
2. 当有 3~10 个 case 数并且 case 的 value 取值非线性的时候，我们可以选择 switch-case 语句;
3. 当 case 数达到 10 个以上并且每次的结果只是一个取值而不是额外的 JavaScript 语句的时候，我们可以选择查找表.
