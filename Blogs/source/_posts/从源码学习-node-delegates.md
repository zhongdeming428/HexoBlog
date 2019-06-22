---
title: 从源码学习 node-delegates
date: 2019-06-22 17:02:09
categories:
  - Nodejs
  - JavaScript
---

[node-delegates](<https://github.com/tj/node-delegates>) 是 [TJ](<https://github.com/tj>) 大神所写的一个简单的小工具，源码只有 157 行，作用在于将外部对象接受到的操作委托到内部属性进行处理，也可以理解为讲对象的内部属性暴露到外部，简化我们所需要书写的代码。

安装和使用的代码在源码仓库都可以找到，这里主要先讲一下 API。

### API

#### Delegate(proto, prop)

用于创建一个 delegator 实例，用于把 proto 接收到的一些操作委托给它的 prop 属性进行处理。

#### Delegate.auto(proto, targetProto, targetProp)

根据 targetProp 所包含的键，自动判断类型，把 targetProto 上的对应属性代理到 proto。可以是 getter、setter、value 或者 method。

#### Delegate.prototype.method(name)

在 proto 对象上新增一个名为 `name` 的函数，调用该函数相当于调用 proto 的 prop 属性上的 name 函数。

#### Delegate.prototype.getter(name)

新增一个 getter 到 proto 对象，访问该 getter 即可访问 proto 的 prop 的对应 getter。

#### Delegate.prototype.setter(name)

同 getter。

<!-- more -->

#### Delegate.prototype.access(name)

在 proto 上同时新增一个 getter 和一个 setter，指向 proto.prop 的对应属性。

#### Delegate.prototype.fluent(name)

`access` 的特殊形式。

```js
delegate(proto, 'request')
  .fluent('query')

// getter
var q = request.query();

// setter (chainable)
request
  .query({ a: 1 })
  .query({ b: 2 });
```

### 源码阅读

```js
/**
 * Expose `Delegator`.
 */

// 暴露 Delegator 构造函数
module.exports = Delegator;

/**
 * Initialize a delegator.
 * 构造一个 delegator 实例
 * @param {Object} proto 外部对象，供外部调用
 * @param {String} target 外部对象的某个属性，包含具体处理逻辑
 * @api public
 */

function Delegator(proto, target) {
  // 如果没有使用 new 操作符调用构造函数，则使用 new 构造
  if (!(this instanceof Delegator)) return new Delegator(proto, target);
  // 构造实例属性
  this.proto = proto;
  this.target = target;
  this.methods = [];
  this.getters = [];
  this.setters = [];
  this.fluents = [];
}

/**
 * Automatically delegate properties
 * from a target prototype
 * 根据 targetProp 自动委托，绑定一个属性到 Delegator 构造函数
 * @param {Object} proto 接受请求的外部对象
 * @param {object} targetProto 处理具体逻辑的内部对象
 * @param {String} targetProp 包含要委托的属性的对象
 * @api public
 */

Delegator.auto = function(proto, targetProto, targetProp){
  var delegator = Delegator(proto, targetProp);
  // 根据 targetProp 获取要委托的属性
  var properties = Object.getOwnPropertyNames(targetProto);
  // 遍历所有要委托的属性
  for (var i = 0; i < properties.length; i++) {
    var property = properties[i];
    // 获取 targetProto 上对应属性的 descriptor
    var descriptor = Object.getOwnPropertyDescriptor(targetProto, property);
    // 如果当前属性的 get 被重写过，就作为 getter 委托（使用 __defineGetter__ 或者 Object.defineProperty 指定 getter 都会重写 descriptor 的 get 属性）
    if (descriptor.get) {
      delegator.getter(property);
    }
    // 同 get，如果 set 被重写过，那就作为 setter 委托
    if (descriptor.set) {
      delegator.setter(property);
    }
    // 如果当前 property 具有 value，那么判断是函数还是普通值
    if (descriptor.hasOwnProperty('value')) { // could be undefined but writable
      var value = descriptor.value;
      if (value instanceof Function) {
        // 是函数就进行函数委托
        delegator.method(property);
      } else {
        // 是普通值就作为 getter 委托
        delegator.getter(property);
      }
      // 如果这个值可以重写，那么继续进行 setter 委托
      if (descriptor.writable) {
        delegator.setter(property);
      }
    }
  }
};

/**
 * Delegate method `name`.
 * 
 * @param {String} name
 * @return {Delegator} self
 * @api public
 */

Delegator.prototype.method = function(name){
  var proto = this.proto;
  var target = this.target;
  this.methods.push(name);

  // 在 proto 上定义一个 name 的方法
  proto[name] = function(){
    // 实际还是调用的 proto[target][name]，内部的 this 还是指向 proto[target]
    return this[target][name].apply(this[target], arguments);
  };

  return this;
};

/**
 * Delegator accessor `name`.
 *
 * @param {String} name
 * @return {Delegator} self
 * @api public
 */

Delegator.prototype.access = function(name){
  // 同时定义 getter 和 setter
  return this.getter(name).setter(name);
};

/**
 * Delegator getter `name`.
 * 委托 name getter
 * @param {String} name
 * @return {Delegator} self
 * @api public
 */

Delegator.prototype.getter = function(name){
  var proto = this.proto;
  var target = this.target;
  this.getters.push(name);

  // 使用 __defineGetter__ 绑定 name getter 到 proto
  proto.__defineGetter__(name, function(){
    // 注意 this 指向 proto 本身，所以 proto[name] 最终访问的还是 proto[target][name]
    return this[target][name];
  });

  // 此处 this 指向 delegator 实例，构造链式调用
  return this;
};

/**
 * Delegator setter `name`.
 * 在 proto 上委托一个 name setter
 * @param {String} name
 * @return {Delegator} self
 * @api public
 */

Delegator.prototype.setter = function(name){
  var proto = this.proto;
  var target = this.target;
  this.setters.push(name);

  // 通过 __defineSetter__ 方法指定一个 setter 到 proto
  proto.__defineSetter__(name, function(val){
    // 注意 this 指向 proto 本身，所以对 proto[name] 设置值即为为 proto[target][name] 设置值
    return this[target][name] = val;
  });

  // 返回自身实现链式调用
  return this;
};

/**
 * Delegator fluent accessor
 *
 * @param {String} name
 * @return {Delegator} self
 * @api public
 */

Delegator.prototype.fluent = function (name) {
  var proto = this.proto;
  var target = this.target;
  this.fluents.push(name);

  proto[name] = function(val){
    // 如果 val 不为空，那么就作为 setter 使用
    if ('undefined' != typeof val) {
      this[target][name] = val;
      // 完事后返回 proto 自身，实现链式调用
      return this;
    } else {
      // 如果 val 未定义，那么作为 getter 使用，返回具体的值
      return this[target][name];
    }
  };

  return this;
};

```

### 具体案例

之所以会研究一下这个库是因为在看 `koa` 源码的时候看到使用了这个库，在 `koa` 中通过使用 `node-delegates` 把 `context.request` 和 `context.response` 上的属性都委托到了 `context` 自身。所以我们可以直接使用 `context.query`、`context.status` 来进行操作，简化了我们所写的代码。

koa 源码位置链接：https://github.com/koajs/koa/blob/b7fc526ea49894f366153bd32997e02568c0b8a6/lib/context.js#L191

### 总结

* 通过 `__defineGetter__` 和 `__defineSetter__` 可以设置 getter 和 setter，但是 MDN 显示这两个 API 已被 deprecated，github 也已经有人提了 issue 和 pr。另外，通过这两个 API 设置 getter 和 setter 时，传递的函数的内部 this 指向原来的属性，比如：
  ```js
  let a = { nickName: 'HotDog' }
  a.__defineGetter__('name', function() {
    return this.nickName // 此处 this 仍然指向 a
  })
  ```
* 学习了委托模式，可以把外部对象接收到的操作委托给内部属性（或其他对象）进行具体的处理。