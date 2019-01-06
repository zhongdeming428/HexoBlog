---
title: React & TypeScript
date: 2019-01-06 13:56:40
categories:
  - React
  - TypeScript
---
之前看了一下 TypeScript 的知识，但是一直没有上手，最近开始结合 React 和 TypeScript 一起尝试了一下，感受还是很好的，所以写一下笔记。

环境配置没有参考其他东西，就是看了下 [Webpack](https://www.webpackjs.com/guides/typescript/) 和 [TypeScript](https://www.tslang.cn/docs/handbook/integrating-with-build-tools.html#webpack) 的官方文档，使用 Webpack 进行构建还是比较简单的。

## 环境构建

创建一个项目目录，然后切换当前目录到项目目录下：

```bash
$ mkdir tsc && cd ./tsc
```

然后使用 npm 初始化项目：

```bash
$ npm init -y
```

然后创建一些项目文件：

```bash
$ mkdir build src
$ touch build/webpack.base.conf.js build/webpack.dev.conf.js build/webpack.prod.conf.js index.html src/index.tsx tsconfig.json
```

接下来，就可以安装一些依赖了：

```bash
$ npm i webpack webpack-cli webpack-merge webpack-dev-server -D
$ npm i html-webpack-plugin clean-webpack-plugin typescript ts-loader style-loader css-loader @types/react @types/react-dom -D
$ npm i react react-dom -S
```

可以注意到我们没有安装 babel 转译器，如果我们只写 `.ts` 或者 `.tsx` 文件，可以不安装 babel。如果要转译处理 `.js` 文件的话，还是要使用到 babel。

我们先写基础配置：

**webpack.base.conf.js**
****

```js
const path = require('path');
const htmlWebpackPlugin = require('html-webpack-plugin');
const cleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
  entry: path.resolve(__dirname, '../src/index.tsx'),
  output: {
    filename: '[name].[hash].js'
  },
  resolve: {
    extensions: ['*', '.js', '.json', '.ts', '.tsx']
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  plugins: [
    new htmlWebpackPlugin({
      inject: true,
      template: path.resolve(__dirname, '../index.html')
    }),
    new cleanWebpackPlugin(['dist'])
  ]
};

```

然后可以构造开发环境下的配置文件：

**webpack.dev.conf.js**
****

```js
const merge = require('webpack-merge');
const path = require('path');
const baseConfig = require('./webpack.base.conf');

module.exports = merge(baseConfig, {
  mode: 'development',
  devtool: 'source-map',
  devServer: {
    port: 9999,
    open: true,
    contentBase: path.resolve(__dirname, '../dist')
  }
});
```

然后添加 npm 脚本到 `package.json` 中：

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "dev": "webpack-dev-server --config ./build/webpack.dev.conf.js"
}
```

然后添加我们的 ts 配置到 `tsconfig.json`：

```js
{
  "compilerOptions": {
    "outDir": "./dist/",  // 打包输出目录
    "noImplicitAny": true,  // 默认必须为变量指定类型
    "module": "es6", // 使用 ESM 模块化方案
    "target": "es5", // 代码编译成 ES 5
    "jsx": "react", // 开启 JSX，使用 react 方式编译，如果要使用 babel 编译，那就将 jsx 设置为 ‘preserve’
    "allowJs": true, // 允许编译 js 代码
    "sourceMap": true,  // 编译后同时产出 map 文件
    "removeComments": true // 移除注释
  }
}
```

更多的配置项解释，参考：[翻译 | 开始使用 TypeScript 和 React](https://juejin.im/post/595cc34ff265da6c3d6c262b)。

写完了以后我们就可以添加内容到我们的开发文件中了：

**index.html**
****

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

**src/index.tsx**
****

```tsx
import * as React from 'react';
import * as ReactDOM from 'react-dom';

ReactDOM.render(
  <h1>Hello TSX!</h1>,
  document.getElementById('root') as HTMLElement
);
```

可以注意到引入 React 和 ReactDOM 的方式和之前有一些不同。
另外由于 TypeScript 的强制转换符 `<>` 和 JSX 的元素相冲突，所以使用 `as` 作为强制转换符。

运行 `npm run dev`，可以查看效果。

使用 TypeScript 以后，项目配置要稍微简单一点。配置好开发环境以后，就可以写代码啦！

## 第一个组件

我以一个 Header 组件为例，效果如下：

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_2019-01-06%2016-11-37%20%e7%9a%84%e5%b1%8f%e5%b9%95%e6%88%aa%e5%9b%be.png)

新建一个 `Header.tsx` 文件和一个 `Header.css` 文件到 `src/components` 下。

由于头部栏的标题文字应该是可以修改的，然后右边的 menu 应该是可以自定义的，所以这些数据应该都可以通过 props 传入我们的 Header 组件。

写我们的 Header 组件：

```tsx
// Header.tsx

// 引入 React
import * as React from 'react';
// 引入我们的组件样式
import './Header.css';

// 定义的接口，用于规范 Header 组件的 props，向外界公开，
// 便于在其他组件中引用时实现这个接口，减少错误
export interface HeaderProps {
  title: string; // 必须给定 title，一个 string 类型的值
  menus?: MenuItemProps[];  // menus 是可选属性，是一个符合 MenuItemProps 接口规范的对象的数组
  height?: string;  // 问号都代表可选项
  bgColor?: string;
}

// 这个接口定义了 MenuItem 组件的 props 规范，同时也定义了
// HeaderProps 中 menus 数组的元素的规范
interface MenuItemProps {
  name: string;  // 给定 menu 的名称
  href: string;  // 给定 menu 要跳转的链接
}

// 定义了 Header 组件的 state 的规范
interface HeaderState {
  isVisible: boolean // 代表 Header 组件是否可见
}

// Header 组件
// 注意 React.Component 后面的泛型，就是我们上方定义的接口，它们分别制定了组件的 props 和 state 的规范
class Header extends React.Component<HeaderProps, HeaderState> {
  // 指定组件实例的 state，必须符合 HeaderState 的规范
  state = {
    isVisible: true
  }
  render() {
    const {
      title,
      menus = [],
      height = '50px',
      bgColor = 'lightblue'
    } = this.props;  // 从 props 获取值，其中可选项都有默认值
    const style = {
      height,
      backgroundColor: bgColor
    };  // 构造 Header 内联样式
    return this.state.isVisible ? <div style={style} className="header">
      <span>{title}</span>
      <div className="header-menus">
        {
          menus.map(item => <MenuItem {...item}/>)
        }
      </div>
    </div> : null;
  }
}

// MenuItem 组件
// state 的规范是一个 object，未指定具体接口类型
class MenuItem extends React.Component<MenuItemProps, object> {
  render() {
    const { name, href } = this.props;
    return <a className="header-menu-item" href={href} key={href}>
      {name}
    </a>
  }
}

// 最后向外部暴露 Header 组件
export default Header;
```

可以看到 TypeScript 结合 React 其实很好用，尤其在规范 props 的时候很好用，能够避免很多编程时候的错误。而 IDE 的提示能够更加地方便我们开发。写起来何止舒服，简直舒服啊～

然后在 `Header.css` 写一下我们的样式：

```css
html,
body,
div {
  margin: 0;
  padding: 0;
  font-size: 16px;
}

.header {
  font-size: 1.5rem;
  line-height: 1rem;
  padding: 1rem;
  box-sizing: border-box;
  position: relative;
}

.header-menus {
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
}

.header-menu-item {
  margin: 0 .5rem;
}
```

这里仅仅是做个示例，排版没有在意太多的通用性，大家看看就好～

然后我们就可以在 `index.tsx` 中使用我们的 Header 组件了：

```tsx
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import Header, { HeaderProps } from './components/Header';  // 引入接口规范和组件


// 构造 Header 组件的 props，必须符合 HeaderProps 接口规范。
// 在写的过程中 IDE 也能给我们很多的提示，方便了开发
const headerProps: HeaderProps = {
  title: 'Hello TSX!',
  menus: [{
    name: 'menu1',
    href: 'https://www.zhongdeming.fun'
  }, {
    name: 'menu2',
    href: 'https://www.baidu.com'
  }],
  bgColor: 'lightyellow'
};

ReactDOM.render(
  <Header {...headerProps}/>,
  document.getElementById('root') as HTMLElement
);

```

这样一个组件就写完了，可以感受到 TypeScript 确实能够加速我们的开发，减少开发中的错误。

下面是一些利用 TypeScript 开发的时候 IDE 给出的提示的截图：

![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_2019-01-06%2015-35-37%20%e7%9a%84%e5%b1%8f%e5%b9%95%e6%88%aa%e5%9b%be.png)
![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_2019-01-06%2015-46-19%20%e7%9a%84%e5%b1%8f%e5%b9%95%e6%88%aa%e5%9b%be.png)
![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_2019-01-06%2015-47-34%20%e7%9a%84%e5%b1%8f%e5%b9%95%e6%88%aa%e5%9b%be.png)
![](https://www.cnblogs.com/images/cnblogs_com/DM428/1358673/o_2019-01-06%2015-48-03%20%e7%9a%84%e5%b1%8f%e5%b9%95%e6%88%aa%e5%9b%be.png)