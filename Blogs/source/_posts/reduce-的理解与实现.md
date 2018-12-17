---
title: Array.prototype.reduce 的理解与实现
date: 2018-12-16 14:29:15
categories:
  - JavaScript
---

Array.prototype.reduce 是 JavaScript 中比较实用的一个函数，但是很多人都没有使用过它，因为 reduce 能做的事情其实 forEach 或者 map 函数也能做，而且比 reduce 好理解。但是 reduce 函数还是值得去了解的。

reduce 函数可以对一个数组进行遍历，然后返回一个累计值，它使用起来比较灵活，下面了解一下它的用法。

reduce 接受两个参数，第二个参数可选：

```js
@param {Function} callback 迭代数组时，求累计值的回调函数
@param {Any} initVal 初始值，可选
```

其中，callback 函数可以接受四个参数：

```js
@param {Any} acc 累计值
@param {Any} val 当前遍历的值
@param {Number} key 当前遍历值的索引
@param {Array} arr 当前遍历的数组
```

<!-- more -->

callback 接受这四个参数，经过处理后返回新的累计值，而这个累计值会作为新的 acc 传递给下一个 callback 处理。直到处理完所有的数组项。得到一个最终的累计值。

reduce 接受的第二个参数是一个初始值，它是可选的。如果我们传递了初始值，那么它会作为 acc 传递给第一个 callback，此时 callback 的第二个参数 val 是数组的第一项；如果我们没有传递初始值给 reduce，那么数组的第一项会作为累计值传递给 callback，数组的第二项会作为当前项传递给 callback。

示例：

对数组求和：

```js
let arr = [1, 2, 3];
let res = arr.reduce((acc, v) => acc + v);
console.log(res); // 6
```

如果我们传递一个初始值：

```js
let arr = [1, 2, 3];
let res = arr.reduce((acc, v) => acc + v, 94);
console.log(res); // 100
```

利用 reduce 求和比 forEach 更加简单，代码也更加优雅，只需要清楚 callback 接受哪些参数，代表什么含义就可以了。

我们还可以利用 reduce 做一些其他的事情，比如对数组去重：

```js
let arr = [1, 1, 1, 2, 3, 3, 4, 3, 2, 4];
let res = arr.reduce((acc, v) => {
  if (acc.indexOf(v) < 0) acc.push(v);
  return acc;
}, []);
console.log(res); // [1, 2, 3, 4]
```

统计数组中每一项出现的次数：

```js
let arr = ['Jerry', 'Tom', 'Jerry', 'Cat', 'Mouse', 'Mouse'];
let res = arr.reduce((acc, v) => {
  if (acc[v] === void 0) acc[v] = 1;
  else acc[v]++;
  return acc;
}, {});
console.log(res); // {Jerry: 2, Tom: 1, Cat: 1, Mouse: 2}
```

将二维数组展开成一维数组：

```js
let arr = [[1, 2, 3], 3, 4, [3, 5]];
let res = arr.reduce((acc, v) => {
  if (v instanceof Array) {
    return [...acc, ...v];
  } else {
    return [...acc, v];
  }
});
console.log(res); // [1, 2, 3, 3, 4, 3, 5]
```

由此可以看出，reduce 函数还是很实用的，但是 reduce 函数兼容性不是特别好，只支持到 IE 9，如果要在 IE 8 及以下使用的话就不行了，所以我们可以自己实现一下，还可以对其做一下扩展，使其能够遍历对象。

首先可以实现一个最基础的 each 函数，作为我们 reduce 的基础：

```js
/**
 * 遍历对象或数组，对操作对象的属性或元素做处理
 * @param {Object|Array} param 要遍历的对象或数组
 * @param {Function} callback 回调函数
 */
function each(param, callback) {
  // ...省略参数校验
  if (param instanceof Array) {
    for (var i = 0; i < param.length; i++) {
      callback(param[i], i, param);
    }
  } else if (Object.prototype.toString.call(param) === '[object Object]') {
    for (var val in param) {
      callback(param[val], val, param);
    }
  } else {
    throw new TypeError('each 参数错误！');
  }
}
```

可以看出 each 可以遍历对象或数组，回调函数接受三个参数：

```js
@param {Any} v 当前遍历项
@param {String|Number} k 当前遍历的索引或键
@param {Object|Array} o 当前遍历的对象或者数组
```

有了这个基础函数，我们可以开始实现我们的 reduce 函数了：

```js
/**
 * 迭代数组、类数组对象或对象，返回一个累计值
 * @param {Object|Array} param 要迭代的数组、类数组对象或对象
 * @param {Function} callback 对每一项进行操作的回调函数，接收四个参数：acc 累加值、v 当前项、k 当前索引、o 当前迭代对象
 * @param {Any} initVal 传入的初始值
 */
function reduce(param, callback, initVal) {
  var hasInitVal = initVal !== void 0;
  var acc = hasInitVal ? initVal : param[0];
  each(hasInitVal ? param : Array.prototype.slice.call(param, 1), (v, k, o) => {
    acc = callback(acc, v, k, o);
  });
  return acc;
}
```

可以看到，我们的 reduce 函数就是在 each 上面封装了一层。根据是否传递了初始值 initVal 来决定遍历的起始项。每次遍历都接受 callback 返回的 acc 值，然后在 reduce 的最后返回 acc 累计值就可以啦！

当然，这部分代码有一个很严重的 bug，导致了我们的 polyfill 毫无意义，那就是遍历对象时的 `for...in`。这个语法和在 IE <= 9 环境下存在 bug，会无法获得对象的属性值，这就导致我们所实现的 reduce 无法在 IE 9 以下遍历对象，但是遍历数组还是可以的。对于 `for...in` 的这个 bug，可以参考 underscore 是怎么实现的，这里暂时不研究了～