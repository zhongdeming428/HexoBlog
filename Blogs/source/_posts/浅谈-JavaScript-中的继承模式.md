---
title: 浅谈 JavaScript 中的继承模式
date: 2018-11-02 21:20:28
categories: JavaScript
---

最近在读一本设计模式的书，书中的开头部分就讲了一下 JavaScript 中的继承，阅读之后写下了这篇博客作为笔记。毕竟好记性不如烂笔头。

JavaScript 是一门面向对象的语言，但是 ES6 之前 JavaScript 是没有类这个概念的。即使 ES6 引入了 class，也只是基于 JavaScript 已有的原型继承模式的一个语法糖，并没有为 JavaScript 引入新的面向对象的继承模型。

但是 JavaScript 是一门非常灵活的语言，为了实现类和继承，JavaScript Developers 已经把原型玩出了花，下面介绍一下 JavaScript 中的继承模式。

JavaScript 中已有的继承模式包括以下六种：

*   类式继承
*   构造函数式继承
*   组合式继承
*   原型式继承
*   寄生式继承
*   寄生组合式继承
<!-- more -->
## 一、类式继承

JavaScript 中，每个对象都有一个原型（prototype），在对象本身找不到对应属性的时候，JavaScript 就会去对象的原型上找，所以这就为 JavaScript 实现继承提供了一种方法 —— 我们只需要把父类的属性放在子类的原型上就行了。因为子类中没有对应属性的话，就会使用原型上的属性，即父类的属性，这就达到了继承的目的。
```js
    // 父类
    function Person() {
        this.name = 'Kevin';
        this.age = 18;
    }
    // 子类
    function Chinese() {
        this.feature = 'hard-working';
    }
    // 指定子类的原型，实现子类继承父类。
    Chinese.prototype = new Person();
    // 实例化子类。
    var c = new Chinese();
    // 输出'hard-working'。
    console.log(c.feature);
    // 输出'Kevin'。
    console.log(c.name);
```
这是一种比较简单直观的继承实现方法，但是存在着问题。所有的子类实例共享了父类实例的属性。如果父类实例的属性是引用类型的值时，将会出现牵一发而动全身的风险。

比如，我们给父类 Person 添加一个 category 的数组属性：
```js
    // 父类
    function Person() {
        this.name = 'Kevin';
        this.age = 18;
        this.category = ['yellow', 'black', 'white'];
    }
    // 子类
    function Chinese() {
        this.feature = 'hard-working';
    }
    // 指定子类的原型，实现子类继承父类。
    Chinese.prototype = new Person();
    // 实例化子类。
    var c1 = new Chinese();
    var c2 = new Chinese();
    // 修改 c1 的 category 属性，世界上还有棕色人种。
    c1.category.push('brown');
    // 输出 c1 和 c2 的 category 属性。
    console.log(c1.category);
    // 结果是 ["yellow", "black", "white", "brown"]
    console.log(c2.category);
    // 结果是 ["yellow", "black", "white", "brown"]
```
我们只修改了 c1 的 category 属性，但是 c2.category 也跟着变化了，因为它们共享了一个 Person 实例，而这个 Person 实例的 category 值是引用类型的。

这就是一个很严重的缺陷了，这样的继承模式会使得我们的数据变得不可预测。

## 二、构造函数式继承

类式继承的致命缺陷是共享引用类型的属性导致牵一发而动全身，而问题的根源在于我们给子类的原型赋值为父类的一个实例，那现在解决这个问题就有办法了。我们可以麻烦一点，不把父类的实例赋值给原型了，直接赋值给每一个子类实例吧，这样就不会存在共享引用类型属性的问题了。
```js
    // 父类
    function Person() {
        this.name = 'Kevin';
        this.age = 18;
        this.category = ['yellow', 'black', 'white'];
    }
    // 子类
    function Chinese() {
        Person.call(this);
        this.feature = 'hard-working';
    }
    // 实例化子类。
    var c1 = new Chinese();
    var c2 = new Chinese();
    // 修改 c1 的 category 属性。
    c1.category.push('brown');
    // 输出 ["yellow", "black", "white", "brown"]
    console.log(c1.category);
    // 输出 ["yellow", "black", "white"]
    console.log(c2.category);
```
可以看到，共享引用类型属性的问题确实解决了。但是这种方法也不太好，因为我们没有利用好原型。每个属性都实例化在实例的本身，造成了资源的浪费，也不符合代码复用的原则。对于一些共享的公用的方法，我们应该绑定在原型上。

## 三、组合式继承

为了解决第二小节的问题，出现了组合式继承。所谓组合式继承，就是综合类式继承和构造函数式继承，结合二者。

比如：
```js
    // 父类
    function Person() {
        this.name = 'Kevin';
        this.age = 18;
        this.category = ['yellow', 'black', 'white'];
    }
    // 子类
    function Chinese() {
        // 基于构造函数式继承。
        Person.call(this);
        this.feature = 'hard-working';
    }
    // 基于类式继承。
    Chinese.prototype = new Person();
    console.log(new Person());
    console.log(new Chinese());
```
这样就“充分地”利用了原型。

但是什么东西都往原型塞，也是一种浪费。我们在子类构造函数中调用了父类的构造函数，在指定子类的原型时又调用了一遍，既不优雅又浪费资源，所以这种方法也不怎么样。

## 四、原型式继承

这是 2006 年道格拉斯.克罗克福德的一篇文章所提出来的，我们把要继承的属性放在原型中，然后再在实例上定义属性，这样就使得每个实例可以有自己特有的属性，还可以和其他实例共享必要的属性。
```js
    function inherit(o) {
        function F(){}
        F.prototype = o;
        return new F();
    }
    var x = inherit({name: 'Kevin'});
    x.age = 18;
    // 输出 {name: 'Kevin', age: 18}
    console.log(x);
```
这种方法首先定义了一个空白的构造函数，然后为其指定了将要继承的属性，然后返回它的一个实例，最后给每个实例指定属性。

这样节约了很多的资源，每个返回的实例也很干净纯洁。

## 五、寄生式继承

这是一种基于原型式继承的方法，只多了一个步骤，就是给每个实例添加属性。这看起来像是第五小节代码的一种封装。
```js
    function inherit(o) {
        function F(){}
        F.prototype = o;
        var f = new F();
        f.name = 'Kevin';
        f.age = 18;
        return f;
    }
    var proto = {
        sayName: function() {
            console.log(this.name);
        }
    };
    var x = inherit(proto);
    // {name: "Kevin", age: 18}
    console.log(x);
    // Kevin
    x.sayName();
```
这种继承方式在 underscore 中曾经使用过，具体哪里我也不记得了，感兴趣的同学可以去看看。

## 六、寄生组合式继承

这是一种综合型的方法，结合了寄生式、组合式继承方法。实际上是寄生式、构造函数式、类式继承的综合体。
```js
    // 基于寄生式继承实现的原型赋值，把子类的原型赋值为父类。
    function inheritProto(SubClass, SuperClass) {
        function F() {}
        F.prototype = SuperClass.prototype;
        var f = new F();
        // 赋值 prototype 会丢失 constructor 指向，重新赋值。
        f.constructor = SubClass;
        SubClass.prototype = f;
    }
    // 父类
    function Person() {
        this.name = 'Kevin';
        this.age = 18;
    }
    Person.prototype.sayName = function() {
        console.log(this.name);
    };
    function Chinese() {
        // 构造函数式继承。
        Person.call(this);
        this.feature = 'hard-working';
        this.name = '李国强';
    }
    // 类式继承原型。
    inheritProto(Chinese, Person);
    var c = new Chinese();
    console.log(c);
    // 输出 {name: "李国强", age: 18, feature: "hard-working"}
    c.sayName();
    // 输出 "李国强"。
```
从注释可以看出来，确实综合了三种继承模式。所以这种方法的名字就叫寄生组合式继承。

笔记一篇，时间很仓促，如果有问题欢迎指正！

所有代码可以复制到浏览器控制台执行查看输出。