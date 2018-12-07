---
title: ES 6 装饰器与 React
date: 2018-12-07 22:04:25
categories:
    - React
---

关于 Decorator 到底是 ES 6 引入的还是 ES 7 引入的我也不是很明白了，两种说法都有，这种问题懒得纠结了……在用的时候发现这个东西很好用，平常用处可能不大，但是结合 React 就很好使了。接下来就讲一讲。

## 一、环境搭建

我搭建了一个 React 开发环境，结合 babel 的插件——`babel-plugin-transform-decorators-legacy`一起使用，这个插件可以让你写 Decorator。

GitHub 地址：`https://github.com/zhongdeming428/HOC`

可以通过如下命令克隆：

```bash
$ git clone https://github.com/zhongdeming428/HOC.git
```
<!-- more -->
克隆下来以后就可以尝试啦！

## 二、Decorator 的基本使用

装饰器本身就是一个函数，使用起来挺简单，无非就是修饰类或者类的函数。使用 `@` 调用，扔在要修饰的类或者类方法前面就可以了。但是在修饰类和类函数的时候又有细微的差异。

```js
class A {
    @sayB
    sayA() {
        console.log('a');
    }
}
function sayB(target, name, descriptor) {
    // ...
}
```

在使用装饰器装饰类函数的时候，可以接受三个参数。第一个是要修饰的对象，第二个是修饰的属性名，第三个是属性描述符。可以在我搭建的项目中进行尝试。

在用装饰器装饰类的时候，只能够接受一个参数——target。这区别于上面的情况：

```js
@APlus
class A {

}
function APlus(target, name, descriptor) {
    // ... 打印一下可以发现 name、descriptor 是 undefined。
}
```

另外，装饰器还可以接受参数，返回一个符合装饰器规范的新函数即可，这样又可以对装饰器的装饰行为进行定制了。比如：

```js
@attach2Prop({ name: 'A' })
class A {

}

@attach2Prop({ name: 'B' })
class B {

}

function attach2Prop(obj) {
    return function(target) {
        target.prototype.$data = obj;
    }
}

console.log((new A()).$data.name);
console.log((new B()).$data.name);
```

结果会输出 `A` 和 `B`。

这就就可以用同一个装饰器实现不同行为的装饰了。

那么结合 React 有什么妙用呢？

## 三、结合 React 使用

### （1）简化 React-Redux 的使用

以往在使用 react-redux 时，在定义好 UI 组件后，还要定义容器组件：

```jsx
class UIComponent extends React.Component {

}

const ContainerComponent = connect(mapState2Props, mapDispatch2Props)(UIComponent);

export default ContainerComponent;
```

有了装饰器之后：

```jsx
@connect(mapState2Props, mapDispatch2Props)
class UIComponent extends React.Component {

}

export default UIComponent;
```

这样用简化的代码达到了同样的效果，还省去了给容器组件命名的麻烦……代码也更加的整洁。

### （2）定制高阶组件

上一小节中的容器组件实际上就是一个高阶组件，但是我们自己有时候也要定义一些高阶组件，实现代码的更高层次的复用。

例如：我们做了一个组件库，里面有一部分的组件是有一个功能特征的，那就是可以拖拽；又比如我们做的移动端组件，需要实现一个左滑删除功能。我们需要给每种具有这个特征的组件写一遍拖拽或者左滑删除逻辑吗？

显然是否定的，我们可以实现一个纯逻辑组件，而非 UI 组件，它的功能就是使得你的 UI 组件具有某种特定功能。比如上面提到的左滑删除或者拖拽。

这个纯逻辑组件就可以是一个装饰器，是一个高阶组件。

在我搭建的开发环境中，就实现了这样一个简单的高阶组件，让你的 UI 组件在鼠标滑入时显示为一只手。

装饰器代码如下：

```jsx
// src/decorators/CursorPointer.js
import React from 'react';

export default Component => class extends React.Component {
  render() {
    return <div style={{cursor: 'pointer', display: 'inline-block'}}>
      <Component/>
    </div>
  }
}
```

这个装饰器（高阶组件）接受一个 React 组件作为参数，然后返回一个新的 React 组件。实现很简单，就是包裹了一层 div，添加了一个 style，就这么简单。以后所有被它装饰的组件都会具有这个特征。

使用这个装饰器：

```jsx
import React from 'react';
import Clickable from '../decorators/CursorPointer';

@Clickable
class ClickablePanel extends React.Component {
  render() {
    return <div className="panel">

    </div>
  }
}

export default ClickablePanel;
```

将装饰器与高阶组件相结合，可以大大优化你的 React 代码！