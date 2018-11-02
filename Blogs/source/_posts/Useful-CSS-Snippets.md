---
title: Useful CSS Snippets
date: 2018-11-02 16:22:03
categories: CSS
---

看了 30 Seconds CSS，有了许多收获，所以写下了这篇文章，算是收藏一些代码小片段，留作后用。

## 一、手写 Loading 动画

### （1）弹性加载动画

CSS 代码如下：
```css
    .bounce-loading {
      width: 20rem;
      height: 10rem;
      background-color:aqua;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .bounce-loading > div {
      width: 1rem;
      height: 1rem;
      border-radius: 0.5rem;
      background-color:blueviolet;
      margin: 0 0.5rem;
      animation: bounce 1s infinite alternate;
    }
    @keyframes bounce {
      0% {
        transform: translateY(0);
        opacity: 1;
      }
      100% {
        transform: translateY(2rem);
        opacity: 0.1;
      }
    }
    .bounce-loading > div:nth-child(2) {
      animation-delay: 0.2s;
    }
    .bounce-loading > div:nth-child(3) {
      animation-delay: 0.4s;
    }
```
HTML 代码如下：
```html
    <div class="bounce-loading">
      <div></div>
      <div></div>
      <div></div>
    </div>
```
效果如下：

![img](https://user-gold-cdn.xitu.io/2018/10/21/1669588f432097e2?w=472&h=229&f=gif&s=66004)
<!-- more --> 
### （2）旋转小圆圈

CSS 代码如下：
```css
    .donut-loading {
      width: 2rem;
      height: 2rem;
      border-radius: 2rem;
      border: 3px solid rgba(0, 0, 0, 0.1);
      border-left-color: #7983ff;
      animation: rotate 1s infinite linear;
    }
    @keyframes rotate {
      from {
        transform: rotate(0deg)
      }
      to {
        transform: rotate(360deg)
      }
    }
```
HTML 代码如下：
```html
    <div class="donut-loading"></div>
```
效果如下：

![img](https://user-gold-cdn.xitu.io/2018/10/21/166958969ce9a9ae?w=89&h=70&f=gif&s=45939)

## 二、构建一个宽高比固定的 div

CSS 代码如下：
```css
    .reactive-height {
      width: 50%;
      background-color: aqua;
    }
    .reactive-height::before {
      content: '';
      float: left;
      padding-top: 100%;
    }
    .reactive-height::after {
      content: "";
      clear: both;
      display: table;
    }
```
HTML 代码如下：
```html
    <div class="reactive-height"></div>
```
## 三、自定义滚动条

CSS 代码如下：
```css
    .custom-scrollbar {
      width: 40rem;
      height: 7rem;
      background-color: aliceblue;
      overflow-y: scroll;
    }
    .custom-scrollbar::-webkit-scrollbar {
      width: 8px;
    }
    .custom-scrollbar::-webkit-scrollbar-thumb {
      border-radius: 10px;
      background-color:mediumpurple;
    }
```
HTML 代码如下：
```html
    <div class="custom-scrollbar">
      <p>
        Pellentesque habitant morbi tristique senectus et 
        netus et malesuada fames ac turpis egestas. 
        Vestibulum tortor quam, feugiat vitae, 
        ultricies eget, tempor sit amet, ante. 
        Donec eu libero sit amet quam egestas semper. 
        Aenean ultricies mi vitae est. Mauris placerat 
        eleifend leo. Quisque sit amet est et sapien 
        ullamcorper pharetra. Vestibulum erat wisi, 
        condimentum sed, commodo vitae, ornare sit amet, 
        wisi. Aenean fermentum, elit eget tincidunt condimentum, 
        eros ipsum rutrum orci, sagittis tempus lacus enim ac dui. 
        Donec non enim in turpis pulvinar facilisis. Ut felis. 
        Praesent dapibus, neque id cursus faucibus, tortor neque 
        egestas augue, eu vulputate magna eros eu erat. Aliquam 
        erat volutpat. Nam dui mi, tincidunt quis, accumsan 
        porttitor, facilisis luctus, metus
      </p>
    </div>
``` 
效果截图如下：

  ![img](https://user-gold-cdn.xitu.io/2018/10/21/1669589eb68aae38?w=991&h=193&f=gif&s=53441)

## 四、自定义文本选择时的样式

CSS 代码如下：
```css
  .custom-text-selection {
    width: 50%;
  }
  .custom-text-selection::selection {
    background-color:navy;
    color: white;
  }
```
HTML 代码如下：
```html
  <p class="custom-text-selection">
    Pellentesque habitant morbi tristique senectus et 
    netus et malesuada fames ac turpis egestas. 
    Vestibulum tortor quam, feugiat vitae, 
    ultricies eget, tempor sit amet, ante. 
    Donec eu libero sit amet quam egestas semper. 
    Aenean ultricies mi vitae est. Mauris placerat 
    eleifend leo. Quisque sit amet est et sapien 
    ullamcorper pharetra. Vestibulum erat wisi, 
    condimentum sed, commodo vitae, ornare sit amet, 
    wisi. Aenean fermentum, elit eget tincidunt condimentum, 
    eros ipsum rutrum orci, sagittis tempus lacus enim ac dui. 
    Donec non enim in turpis pulvinar facilisis. Ut felis. 
    Praesent dapibus, neque id cursus faucibus, tortor neque 
    egestas augue, eu vulputate magna eros eu erat. Aliquam 
    erat volutpat. Nam dui mi, tincidunt quis, accumsan 
    porttitor, facilisis luctus, metus
  </p>
```
效果截图如下：

![img](https://user-gold-cdn.xitu.io/2018/10/21/166958a71843a806?w=1007&h=352&f=gif&s=62142)

## 五、禁止文本被选中

CSS 代码如下：
```css
    .disable-selection {
      width: 50%;
      user-select: none;
    }
```
HTML 代码如下：
```html
    <p class="disable-selection">
      Pellentesque habitant morbi tristique senectus et 
      netus et malesuada fames ac turpis egestas. 
      Vestibulum tortor quam, feugiat vitae, 
      ultricies eget, tempor sit amet, ante. 
      Donec eu libero sit amet quam egestas semper. 
      Aenean ultricies mi vitae est. Mauris placerat 
      eleifend leo. Quisque sit amet est et sapien 
      ullamcorper pharetra. Vestibulum erat wisi, 
      condimentum sed, commodo vitae, ornare sit amet, 
      wisi. Aenean fermentum, elit eget tincidunt condimentum, 
      eros ipsum rutrum orci, sagittis tempus lacus enim ac dui. 
      Donec non enim in turpis pulvinar facilisis. Ut felis. 
      Praesent dapibus, neque id cursus faucibus, tortor neque 
      egestas augue, eu vulputate magna eros eu erat. Aliquam 
      erat volutpat. Nam dui mi, tincidunt quis, accumsan 
      porttitor, facilisis luctus, metus
    </p>
```
## 六、渐变色文本

HTML 代码如下：
```html
    <p class="gradient-text">
      gradient-text
    </p>
```
CSS 代码如下：
```css
    .gradient-text {
      background: -webkit-linear-gradient(pink, red);
      -webkit-text-fill-color: transparent;
      -webkit-background-clip: text;
    }
```
效果截图如下：

![img](https://user-gold-cdn.xitu.io/2018/10/21/166958ac186ecaa4?w=615&h=147&f=gif&s=5740)

## 七、Hover 下划线效果

该部分实现一个鼠标移入时的下划线变化效果，共用一段 HTML 代码，代码如下：
```html
    <p class="hover-underline-animation">
      Hover Underline Animation
    </p>
```
各部分实现效果的 CSS 代码各异，将分别给出。

### （1）从中间开始变化

CSS 代码如下：
```css
    .hover-underline-animation {
      cursor: pointer;
    }
    .hover-underline-animation::after {
      content: '';
      width: 100%;
      height: 2px;
      display: block;
      background-color: #7983ff;
      transform: scaleX(0);
      transition: transform 0.3s;
    }
    .hover-underline-animation:hover::after {
      transform: scaleX(1);
    }
```
效果截图如下：

  ![img](https://user-gold-cdn.xitu.io/2018/10/21/166958b6988221d8?w=528&h=79&f=gif&s=19156)

### （2）从左至右变化

CSS 代码如下：
```css
    .hover-underline-animation {
      cursor: pointer;
    }
    .hover-underline-animation::after {
      content: '';
      width: 100%;
      height: 2px;
      display: block;
      background-color: #7983ff;
      transform: scaleX(0);
      transform-origin: right;
      transition: transform 0.3s;
    }
    .hover-underline-animation:hover::after {
      transform: scaleX(1);
      transform-origin: left;
    }
```
效果截图如下：

![img](https://user-gold-cdn.xitu.io/2018/10/21/166958bd76246b68?w=528&h=79&f=gif&s=17102)

### （3）实现左入左出、右入右出的效果

这一部分 HTML 代码略有不同，为了展示左入左出、右入右出的效果，需要三个元素来实现，所以 HTML 代码多了两个相同的元素：
```html
    <span class="hover-underline-animation">
      Hover Underline Animation
    </span>
    <span class="hover-underline-animation">
      Hover Underline Animation
    </span>
    <span class="hover-underline-animation">
      Hover Underline Animation
    </span>
```
CSS 代码如下;
```css
    .hover-underline-animation {
      cursor: pointer;
      position: relative;
    }
    .hover-underline-animation::after {
      content: '';
      position: absolute;
      right: 0;
      bottom: 0;
      width: 0%;
      height: 2px;
      display: block;
      background-color: #7983ff;
      transition: all 0.3s;
    }
    .hover-underline-animation:hover::after {
      width: 100%;
    }
    .hover-underline-animation:hover ~ .hover-underline-animation::after {
      right: 100% !important;
    }
```
效果截图如下：

![img](https://user-gold-cdn.xitu.io/2018/10/21/166958c2b4b0426d?w=1000&h=75&f=gif&s=67570)

## 八、:not 选择器

HTML 代码如下：
```html
    <ul class="not-selector" type="none">
      <li>One</li>
      <li>Two</li>
      <li>Three</li>
      <li>Four</li>
    </ul>
```
CSS 代码如下：
```css
    .not-selector > li {
      width: 20rem;
      position: relative;
    }
    .not-selector > li:not(:last-child)::after {
      content: "";
      display: inline-block;
      background-color: #c3c3c3;
      height: 0.5px;
      width: 100%;
      position: absolute;
      bottom: 0;
      left: 0;
    }
```
实现效果如下：

![img](https://user-gold-cdn.xitu.io/2018/10/21/166958cb638629e1?w=567&h=178&f=gif&s=2971)

## 九、滚动容器的渐变遮罩

HTML 代码如下：
```html
    <div class="overflow-scroll-gradient dn">
      <div>
        Pellentesque habitant morbi tristique senectus et 
        netus et malesuada fames ac turpis egestas. 
        Vestibulum tortor quam, feugiat vitae, 
        ultricies eget, tempor sit amet, ante. 
        Donec eu libero sit amet quam egestas semper. 
        Aenean ultricies mi vitae est. Mauris placerat 
        eleifend leo. Quisque sit amet est et sapien 
        ullamcorper pharetra. Vestibulum erat wisi, 
        condimentum sed, commodo vitae, ornare sit amet, 
        wisi. Aenean fermentum, elit eget tincidunt condimentum, 
        eros ipsum rutrum orci, sagittis tempus lacus enim ac dui. 
        Donec non enim in turpis pulvinar facilisis. Ut felis. 
        Praesent dapibus, neque id cursus faucibus, tortor neque 
        egestas augue, eu vulputate magna eros eu erat. Aliquam 
        erat volutpat. Nam dui mi, tincidunt quis, accumsan 
        porttitor, facilisis luctus, metus
      </div>
    </div>
```
CSS 代码如下：
```css
    .overflow-scroll-gradient {
      position: relative;
    }
    .overflow-scroll-gradient::before {
      content: "";
      display: inline-block;
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 5rem;
      background: linear-gradient(rgba(255, 255, 255, 1), rgba(255, 255, 255, 0.001))
    }
    .overflow-scroll-gradient::after {
      content: "";
      display: inline-block;
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 5rem;
      background: linear-gradient(rgba(255, 255, 255, 0.001), rgba(255, 255, 255, 1))
    }
    .overflow-scroll-gradient > div {
      width: 15rem;
      height: 25rem;
      overflow-y: scroll;
    }
```
效果截图如下：

![img](https://user-gold-cdn.xitu.io/2018/10/21/166958ce71ae969c?w=751&h=635&f=gif&s=663102)

## 十、使用系统字体获得原生体验

HTML 代码：
```html
    <p class="system-font-stack">This text uses the system font.</p>
```
CSS 代码如下：
```css
    .system-font-stack {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen-Sans, Ubuntu, Cantarell, 'Helvetica Neue', Helvetica, Arial, sans-serif;
    }
```
我在 Ubuntu 系统下显示效果如下：

![img](https://user-gold-cdn.xitu.io/2018/10/21/166958d4832c8669?w=905&h=194&f=png&s=11999)

## 十一、圆润的 checkbox

HTML 代码如下：
```html
    <div>
      <input type="checkbox" id="toggle" class="offscreen">
      <label for="toggle" class="checkbox"></label>
    </div>
```
CSS 代码如下：
```css
    .offscreen {
      display: none;
    }
    .checkbox {
      width: 40px;
      height: 20px;
      border-radius: 20px;
      display: inline-block;
      background-color: rgba(0, 0, 0, 0.25);
      position: relative;
      cursor: pointer;
    }
    .checkbox::before {
      content: "";
      width: 18px;
      height: 18px;
      border-radius: 18px;
      background-color: white;
      position: absolute;
      left: 1px;
      top: 1px;
      transition: transform .3s ease;
    }
    #toggle:checked + .checkbox {
      background-color: #7983ff;
    }
    #toggle:checked + .checkbox::before {
      transform: translateX(20px);
    }
```
效果截图如下：

![img](https://user-gold-cdn.xitu.io/2018/10/21/166958d8c308a316?w=246&h=78&f=gif&s=19864)

## 十二、绘制一个三角形

HTML 代码如下：
```html
    <div class="triangle"></div>
```
CSS 代码如下：
```css
    .triangle {
      width: 0;
      height: 0;
      border: 1rem solid transparent;
      border-bottom: 3rem solid blue;
    }
```
利用 CSS border 的特性绘制三角形，改变 border 的宽度，可以绘制不同特性的三角形。

## 十三、过长的文本用省略号代替

HTML 代码如下：
```html
    <p class="truncate-text">
      This text will be truncated with ellipse ......
    </p>
```
CSS 代码如下：
```css
    .truncate-text {
      width: 19rem;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      background-color: #c3c3c3
    }
```
效果截图如下;

![img](https://user-gold-cdn.xitu.io/2018/10/21/166958e07efa6fc6?w=722&h=115&f=png&s=9232)