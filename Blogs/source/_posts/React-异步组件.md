---
title: React 异步组件
date: 2019-01-12 13:13:09
categories:
  - React
  - React-Router
---

之前写过一篇 [Vue 异步组件的文章](https://github.com/zhongdeming428/Blog/issues/27)，最近在做一个简单项目的时候又想用到 React 异步组件，所以简单地了解了一下使用方法，这里做下笔记。

传统的 React 异步组件基本都靠自己实现，自己写一个专门的 React 组件加载函数作为异步组件的实现工具，通过 `import()` 动态导入，实现异步加载，可以参考[【翻译】基于 Create React App路由4.0的异步组件加载（Code Splitting）](https://segmentfault.com/a/1190000010067597)这篇文章。这样做的话还是要自己写一个单独的加载组件，有点麻烦。于是想找个更简单一点的方式，没想到真给找到了：[Async React using React Router & Suspense](https://itnext.io/async-react-using-react-router-suspense-a86ade1176dc)，这篇文章讲述了如何基于 React Router 4 和 React 的新特性快速实现异步组件按需加载。

2018 年 10 月 23 号，React 发布了 [v16.6 版本](https://react.docschina.org/blog/2018/10/23/react-v-16-6.html)，新版本中有个新特性叫 `lazy`，通过 [lazy](https://react.docschina.org/blog/2018/10/23/react-v-16-6.html#reactlazy-code-splitting-with-suspense) 和 Suspense 组件我们就可以实现异步组件，如果你使用的是 React v16.6 以上版本：

<!-- more -->

最简单的实现方法：

```jsx
// codes from https://react.docschina.org

import React, { lazy, Suspense } from 'react';
const OtherComponent = lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <OtherComponent />
    </Suspense>
  );
}
```

从 React 中引入 `lazy` 方法和 `Suspense` 组件，然后用 lazy 方法处理我们的组件，lazy 会返回一个新的React 组件，我们可以直接在 Suspense 标签内使用，这样组件就会在匹配的时候才加载。

lazy 接受一个函数作为参数，函数内部使用 `import()` 方法异步加载组件，加载的结果返回。

Suspense 组件的 `fallback` 属性是必填属性，它接受一个组件，在内部的异步组件还未加载完成时显示，所以我们通常传递一个 `Loading` 组件给它，如果没有传递的话，就会报错。

所以在使用 React Router 4 的时候，我们可以这样写：

```jsx
import React, { lazy, Suspense } from 'react';
import { HashRouter, Route, Switch } from 'react-router-dom';

const Index = lazy(() => import('components/Index'));
const List = lazy(() => import('components/List'));

class App extends React.Component {
  render() {
    return <div>
      <HashRouter>
        <Suspense fallback={Loading}>
          <Switch>
            <Route path="/index" exact component={Index}/>
            <Route path="/list" exact component={List}/>
          </Switch>
        </Suspense>
      </HashRouter>
    </div>
  }
}

function Loading() {
  return <div>
    Loading...
  </div>
}

export default App;
```

在某些 React 版本中，lazy 函数还有 bug，会导致 React Router 的 component 属性接受 lazy 函数返回结果时报错：[React.lazy makes Route's proptypes fail](https://github.com/ReactTraining/react-router/issues/6420)。

我也遇到了这种 bug，具体的依赖版本如下：

```json
"react": "^16.7.0",
"react-dom": "^16.7.0",
"react-router-dom": "^4.3.1"
```
首次安装依赖后就再也没有更新过，所以小版本应该也是上面的小版本，不存在更新。

解决方法可以把 lazy 的结果放在函数的返回结果中：

```jsx
import React, { lazy, Suspense } from 'react';
import { HashRouter, Route, Switch } from 'react-router-dom';

const Index = lazy(() => import('components/Index'));
const List = lazy(() => import('components/List'));

class App extends React.Component {
  render() {
    return <div>
      <HashRouter>
        <Suspense fallback={Loading}>
          <Switch>
            <Route path="/index" exact component={props => <Index {...props}/>}/>
            <Route path="/list" exact component={props => <List {...props}/>}/>
          </Switch>
        </Suspense>
      </HashRouter>
    </div>
  }
}

function Loading() {
  return <div>
    Loading...
  </div>
}

export default App;
```

上面代码和之前唯一的不同就是把 lazy 返回的组件包裹在匿名函数中传递给 Route 组件的 component 属性。

这样我们的组件都会在路由匹配的时候才开始加载，Webpack 也会自动代码进行 code split，切割成很多小块，减小了首页的加载时间以及单独一个 js 文件的体积。在工作中已经实践过了，确实好用：

![pic](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_%e6%b7%b1%e5%ba%a6%e6%88%aa%e5%9b%be_google-chrome_20190112142942.png)

如果没有使用 React v16.6 以上版本，也可以自己实现，我们可以写一个专门用于异步加载的函数：

```jsx
function asyncComponent(importComponent) {
  class AsyncComponent extends React.Component {
    render() {
      return this.state.component;
    }
    state = {
      component: null
    }
    async componentDidMount() {
      const { default: Component } = await importComponent();
      this.setState({
        component: <Component/>
      });
    }
  }
  return AsyncComponent;
}
```

使用的方法与 `React.lazy` 相同，传入一个异步加载的函数即可，上面这个函数需要注意的地方就是 `import()` 进来的组件被包裹在 default 属性里，结构时要用 `const { default: Component } = ...` 这种形式。

效果如下：

![pic](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_%e6%b7%b1%e5%ba%a6%e6%88%aa%e5%9b%be_google-chrome_20190112145737.png)

总的来说：

* 新版 React 使用起来更加简便～
* 异步组件按需加载这些操作都是基于打包工具的特性，比如 Webpack 的 `import` ～