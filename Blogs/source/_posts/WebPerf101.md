---
title: 常见 Web 性能优化方式
date: 2019-05-06 23:39:01
categories:
    -   Web 性能优化
---

> 这篇文章是我阅读 [Web Performance 101](<https://3perf.com/talks/web-perf-101/>) 之后的进行的粗糙的翻译作为笔记，英语还行的童鞋可以直接看原文。

这篇文章主要介绍了现代 web 加载性能（注意不涉及代码算法等），学习为什么加载性能很重要、有哪些优化的方法以及有哪些工具可以帮助我们对网站进行优化。

### 为什么性能优化很重要？

![img](https://3perf.com/static/perf-importance-horror-9c140b96c24a907753f99b08a36d5556-ea61a.png)

首先，加载缓慢的网站让人很不舒服！

最明显的例子就是当一个移动网站加载太慢的时候，用户体验如同观看一部恐怖电影。

<!-- more -->

图片来源： [Luke Wroblewski](https://twitter.com/lukew/status/846440789752242176?lang=en)

![img](https://3perf.com/static/perf-importance-digits-1e12a65cb66bd47b2745e103a9f63d37.svg)

第二，网站性能直接影响你的产品质量。

—— 2016 年，AliExpress 将他们网站的性能提升了三分之一，然后他们收到的订单增加了 10.5%！

——2006 年，谷歌曾经尝试将他们的搜索放慢 0.5 秒然后发现用户的搜索（请求）次数减少了 25%。

——2008 年，Aberdeen 集团发现将网站放慢 1s，会导致用户满意度下降 16%。

此外还有一系列如上的数据，不管是新的还是旧的：([wpostats.com](https://wpostats.com/) · [pwastats.com](https://pwastats.com/))。

这就是为什么网站性能很重要。

![img](https://3perf.com/static/perf-importance-parts-1-1eb7bf92af22891c83756f425e27c852-ea61a.png)

现在，我们需要弄懂当我们说一个网站很快意味着什么。

在什么情况下可以说一个网站很快？

——它必须加载很快（文件下载、界面渲染），

——然后，在加载之后，它必须很快的执行（比如动画不跳帧、滚动很丝滑）。

![img](https://3perf.com/static/perf-importance-parts-2-6472d2e0d5cd3b049d9d3d9ceeb343cc-ea61a.png)

网站加载很快意味着：

——服务器对于客户端请求响应很快，

——网站自身加载渲染很快。

![img](https://3perf.com/static/perf-importance-parts-3-46caac2ad010e86520da214e91922bc7-ea61a.png)

在这篇文章中，我们将会讨论这个因素：如何让网站快速加载以及渲染。

### 有哪些性能优化方式？

#### JavaScript

##### 一、压缩代码

先从 JavaScript 开始吧。通常情况下，JavaScript 是网站加载缓慢的根源。

![img](https://3perf.com/static/javascript-minification-source-1dd70ff2f2185965e3ce40e0882bc589-ea61a.png)

第一种 JavaScript 优化方式是压缩，如果你已经知道了的话，直接跳过吧。

什么是压缩？在一般情况下，人们写 JavaScript 代码会使用一种方便的格式，包含缩进、富有含义的长变量名、写注释等等。因为这种方式，代码具有很高的可读性，但是多余的空格和注释会使得 JavaScript 文件变得很大。

![img](http://3perf.com/static/javascript-minification-result-da7ff9f731c4433b079ea567b5f884f7-ea61a.png)

为了解决这个问题，人们想到了代码压缩。在压缩的过程中，代码会被去掉所有不必要的字母，替换成短的变量名，去掉注释等等。在最后，代码文件变得比之前更小，但是代码的功能并不受影响。

代码压缩可以将代码文件减小大约 30% ～ 40%。

![img](https://3perf.com/static/javascript-minification-tools-59e14519a11ce0b4c909805c54226b49-ea61a.png)

主流的代码打包工具都支持代码压缩：

—— [`mode: production`](https://webpack.js.org/concepts/mode/) in webpack, 

—— [`babel-preset-minify`](https://www.npmjs.com/package/babel-preset-minify) in Babel,

—— [`gulp-uglify`](https://www.npmjs.com/package/gulp-uglify) in Gulp

##### 二、使用 `async` 和 `defer`

![img](https://3perf.com/static/js-download-1-aa83a6e91c7988bbb3e3f7ca941baa8d-ea61a.png)

接下来，你写了一个 JavaScript 脚本，然后进行了压缩，现在想要在页面中加载它。该如何做呢？

![img](https://3perf.com/static/js-download-2-2c7186bcdd7586baf27163786e3a9c22-ea61a.png)

最简单的方式就是写一个 script 标签，然后 src 属性指向你所写脚本的路径，然后它就可以照常开始工作啦！

但是，你知道这种方法有什么问题吗？

![img](https://3perf.com/static/js-scripts-block-parsing-1-0c1170e780a80fba26c88d570f4eb332.svg)

问题就在于 JavaScript 会阻塞渲染。

![img](https://3perf.com/static/js-scripts-block-parsing-2-45e62a7a6b4dc91b6c0127b0391315d4-ea61a.png)

这是什么意思？

当你的浏览器加载页面的时候，它会转换 HTML 文档成为标签，然后构建 DOM 树。随后它会使用 DOM 树渲染页面。

问题在于，JavaScript 代码可以改变 DOM 树的构建方式。

![img](https://3perf.com/static/js-scripts-block-parsing-3-9bb98754bf1ddd1a315b170ef117aaed-ea61a.png)

例如，JavaScript 可以通过 document.write 写一个 HTML 注释的起始标签到文档中，然后整个 DOM 树都会被毁掉。

 这就是为什么浏览器在碰到 script 标签的时候会停止渲染页面，这样做可以防止 document 做多余的工作。

![img](https://3perf.com/static/js-scripts-block-parsing-4-95dbcf802dc5261d1ceee8fa9114e06a-ea61a.png)

从浏览器的角度来看：

——浏览器遍历文档，然后会解析它

——在某些时刻，浏览器遇到了 script 标签，然后停止了 HTML 转换，它开始下载并执行那些 script 代码

——一旦代码执行完毕，浏览器继续遍历 HTML 文档，然后渲染页面

![img](https://3perf.com/static/js-scripts-block-parsing-5-4160cb99dcbf91efeb902c2cd293c3e9-ea61a.png)

实际上，这意味着当你添加一个 script 标签到页面中时，它后面的内容在它下载并执行完毕之前都是不可见的。如果你添加一个 script 到 head 标签中，所有的内容都会变得不可见——直到 script 被下载执行完毕。

![img](https://3perf.com/static/js-async-defer-1-504bbf2780acf2a967c8fcd1991e69fd-ea61a.png)

那我们该怎么办呢？应该使用 `async` 和 `defer` 属性。

这些属性让浏览器直到 script 脚本可以在后台下载，不必阻塞文档渲染，下面是详细的介绍：

——`async` 让浏览器异步下载（在后台）script 代码，然后继续解析渲染 HTML。（如果在页面渲染完毕之前，script 代码已经下载好了，那么就先停止渲染，先执行 script  代码。由于下载所消耗的时间通常大于 HTML 转化，所以这种情况实际上不多见）。

——`defer `会告诉浏览器在后台异步下载 script 代码，直到 HTML 转化渲染完毕才开始执行这些 script 代码。

![img](https://3perf.com/static/js-async-defer-2-05e8c54d97c4679e58db5b8297e0ffea-ea61a.png)

这里有两大不同点：

——`async` script 标签会在下载之后尽快地执行，它们的执行顺序没有规律。这就意味着有 async 属性的 React bundle script 和 app bundle script 在同一时刻开始下载，由于 app bundle 更小所以会先下载完毕，导致 app 的 bundle script 先执行。然后网站就崩掉了～

——`defer` 不像 `async`，会在加载以及文档渲染完毕之后按照 script 标签的顺序开始执行，因此，`defer` 是更适合的优化方案。

![img](https://3perf.com/static/js-async-defer-3-8b801529ac4a0b76e96ca3e05d51b121-ea61a.png)

##### 三、代码切割

![img](https://3perf.com/static/js-code-splitting-1-349a79f9558606f51a3e6d16687b522f-ea61a.png)

继续。

很多时候，应用都是打包到一个 bundle 里面，然后每次请求都发送到客户端。但是这样做的问题在于有些页面我们见到的场景很少，但是它们的代码同样被打包到了我们的 bundle 中，这样每次页面加载的代码多于实际需要，造成了性能浪费。

![img](https://3perf.com/static/js-code-splitting-3-33849421e63178b77f9bf02bf47c7145-ea61a.png)

这个问题通常使用代码切割进行解决，把大的 bundle 切割成一个个小的。

通过代码切割，我们把不同功能的代码打包到了不同的文件，只在必要的时候加载必要的代码。由于使用这样的做法，用户再也不会下载他们不需要用到的代码了。

![img](https://3perf.com/static/js-code-splitting-3-2-13d6f6ac3e4da1e715bb74d6e41d3279-ea61a.png)

那么我们怎么切割代码呢？

首先，你需要一个代码打包工具，比如 Webpack、Parcel 或者 Rollup。所有的这几个工具都支持一个特殊函数 `import()`。

在浏览器中，`import()` 接受传递给它的 JS 文件并异步下载该文件。这可以用于加载应用程序一开始不需要但是接下来可能会用到的库。

![img](https://3perf.com/static/js-code-splitting-4-aedbcd527d8c06a9db0c1a6f62ff3ea2-ea61a.png)

但是在打包工具中，`import()` 的功能又有所不同。如果你在代码中传递了一个文件给 `import()` 并且在之后进行打包，打包工具会把这个文件以及其所有的依赖打包到一个单独的文件中。app 运行到 import 函数时会单独下载这个文件。

因此，在上方的例子中，webpack 会把 `ChangeAvatarModal.js` 及其依赖打包到单独文件中。在代码执行到 import 时，这个单独文件会被下载。

这就是实际的代码切割。

![img](https://3perf.com/static/js-code-splitting-5-baa4c255eb1a4a9c960e5a8c2d88e9cc-ea61a.png)

第二，在 React 和 Vuejs 中，会有基于 `import()` 的工具能够让你的代码切割工作更加轻松。

例如，[`react-loadable`](https://github.com/jamiebuilds/react-loadable) 是一个组件，用于等待其他组件加载，在其他组件加载时，它会进行占位。React 16.6 添加了一个相似的内置功能组件，叫做 [`Suspense`](https://reactjs.org/blog/2018/10/23/react-v-16-6.html#reactlazy-code-splitting-with-suspense)。此外 Vuejs 也已经支持异步组件一段时间了。

![img](https://3perf.com/static/js-code-splitting-6-437ce516327528c2ad22088eb099b034-ea61a.png)

如果优化得很好的话，我们可以减少很多不必要的数据的下载，代码切割能够成为最重要的流量优化工具。

如果你的 app 只能做一种优化的话，那就是代码切割。

##### 四、移除依赖中的未使用代码

![img](https://3perf.com/static/js-unused-dependencies-1-7d74b5234dd51b811760a9ce088a1b85-ea61a.png)

另外一个重要的优化点在于包的依赖。

——例如，momentjs 这个库，用于进行时间操作，它包含了大约 160 kb 大小的不同语言的文件包。

——React 甚至把 `propTypes` 包含在生产环境的包中，尽管这不是必要的。

——Lodash，你很有可能引入了整个完整的包，尽管你可能只需要其中的一两个方法。

上面这些就是把不必要的代码引入打包的情况。

![img](https://3perf.com/static/js-unused-dependencies-2-8e3f05b12108dea1c09b4a8925e7d311-ea61a.png)

为了帮助开发者移除多余的代码，作者和谷歌一起维护了一个 repo 收集关于如何在 webpack 中优化你的依赖，使用这些建议可以让你的 app 更快更轻巧！

→ [GoogleChromeLabs/webpack-libs-optimizations](https://github.com/GoogleChromeLabs/webpack-libs-optimizations)

##### 五、总结

![img](https://3perf.com/static/js-summing-up-83ea1a6f7d30fd02ca7431486a479727-ea61a.png)

以上都是 JavaScript 的优化方式，总结起来就是：

——压缩你的 js 代码

——使用 `async` 和 `defer` 加载 script

——切割你的代码，让应用只加载必须的代码

——移除依赖中实际未使用的代码

#### CSS

接下来是如何优化 css 代码。

![img](https://3perf.com/static/css-minify-2-56ba1a32ad3fa285056b0e88e3f8cc81-ea61a.png)

##### 一、压缩 CSS 代码

首先，压缩 CSS，就像 JavaScript 代码一样。删除不必要的空格和字母来使你的代码更小。

![img](https://3perf.com/static/css-minify-3-58b79b30d8bc97d9be211d09e47a855f-ea61a.png)

这些工具可以帮助你压缩 CSS 代码：

—— webpack’s [`postcss-loader`](https://github.com/postcss/postcss-loader) with [`cssnano`](https://github.com/cssnano/cssnano) 

—— PostCSS’s [`cssnano`](https://github.com/cssnano/cssnano) 

—— Gulp’s [`gulp-clean-css`](https://www.npmjs.com/package/gulp-clean-css)

##### 二、提取 Critical CSS

![img](https://3perf.com/static/css-block-rendering-1-f92a96c8cf3a91d943e57d9785e99018-ea61a.png)

第二、styles 阻塞渲染，就像之前 script 那样。

![img](https://3perf.com/static/css-block-rendering-2-690e89e7c107c219154c08d461def900-ea61a.png)

因为没有样式的网站看起来很奇怪。

如果浏览器在样式加载之前渲染页面，那么用户就会看到上面那样的情况。

![img](https://3perf.com/static/css-block-rendering-3-f6613023189d8b5fa72b7246a1830e60-ea61a.png)

然后页面就会闪烁，然后就会看到上面截图这样子，很难说是一种好的用户体验。

![img](https://3perf.com/static/css-block-rendering-4-4f72853af496b8c076d223d69bb7bc2f-ea61a.png)

这就是为什么样式加载的时候页面会是空白的。

现在有一种比较机智的优化方式。浏览器在加载样式之前保持空白页是很有理由的，我们不必从这一点下手。但是我们仍然可以想办法让页面渲染更快——让页面只加载渲染初始界面所必要的样式，剩余的样式在之后加载，这些渲染初始界面所必要的样式称为“Critical CSS”。

让我们看看是怎么做的。

![img](https://3perf.com/static/css-critical-2-1-358ef8e3c21188fedfc88b60d3bab555-ea61a.png)

1、把页面样式分为 critical 的和 non-critical 的。

2、把 critical CSS 嵌入到 HTML，真能够让它们尽快地被加载。

![img](https://3perf.com/static/css-critical-2-2-aa2dd94381dd8f78cb5eb4cbed41be09-ea61a.png)

现在，当你加载页面的时候，页面能够很快地被渲染，但是你仍然得加载那些不重要的 CSS。

有多种方式可以加载剩余的 CSS，下面的方式是我所倾向的：

3、使用[`<link rel="preload">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content) 获取非必要的 CSS。

4、___一旦文件被加载到缓存以后，把 `rel` 属性从 `preload` 切换为 `stylesheet`。这可以让浏览器从缓存中获取 CSS 并应用到页面中___。

![img](https://3perf.com/static/css-critical-1-249cde60f6ad2a0170d6bd9e715869bb-ea61a.png)

那我们怎么知道哪些 CSS 是必须的，哪些 CSS 是不必须的呢？通常情况下，规则如下：

```
移除 CSS 样式知道页面看起来变得滑稽，那么剩下的 CSS 就是必要的。
```

例如，页面的布局样式或者文章的文本样式是必须的，因为缺少它们会使得页面看起来很烂。而 JavaScript 弹出窗或者页脚的样式是非必须的，因为用户不会在一开始就看到它们，缺少那些样式，页面看起来仍然十分完美。

![img](https://3perf.com/static/css-critical-7-daaadf57311a8bb395914d226f0a2f9d-ea61a.png)

听起来可能比较复杂，但是有很多自动化工具可以帮助我们完成这项工作。

—— [`styled-components`](https://github.com/styled-components/styled-components). It’s a CSS-in-JS library that extracts and returns critical styles during server-side rendering. It works only if you already write styles using `styled-components`, but if you do, it works really well. 

——[`critical`](https://github.com/addyosmani/critical). It’s a utility that takes an HTML page, renders it in a headless browser and extracts styles for above-the-fold content. Because `critical` runs only over a single page, it might not work well for complex single-page apps.

—— [`penthouse`](https://github.com/pocketjoso/penthouse). It’s similar to `critical` but works with URLs instead of HTML pages.

![img](https://3perf.com/static/css-critical-5-7c585247f4d7ebbfbd6522a07a9e3057-ea61a.png)

这种做法一般可以节约 200 ～ 500 ms 左右的首屏渲染时间。

了解更多 Critical CSS 的知识，阅读 [the amazing Smashing Magazine’s guide](https://www.smashingmagazine.com/2015/08/understanding-critical-css/).

##### 三、总结

![img](https://3perf.com/static/css-summing-up-ea1f22072070c7055c672163245b3e9b-ea61a.png)

这就是 CSS 优化的主要策略，总结起来就是：

——压缩 CSS 代码

——提取必要的 CSS，让页面首先加载它们

#### HTTP

现在让我们看看 HTTP 的优化。

![img](https://3perf.com/static/http-html-minify-2-8daf736ae2c0f18a0f3a3791903f5b2e-ea61a.png)

##### 一、压缩代码

让 HTTP 传输较少数据的方式仍然是压缩代码，本节主要说压缩 HTML 代码，JS、CSS 的代码压缩在之前已经讲过了。

##### 二、GZIP 压缩

![img](https://3perf.com/static/http-gzip-1-90c3e41c725362b7331aaf946a8f05d9-ea61a.png)

压缩代码的第二种方式是 GZIP 压缩。

Gzip 是一种算法，它可以使用复杂的归档算法压缩你发送到客户端的数据。在压缩之后，你的文件看起来像是无法打开的二进制文件，但是它们的体积会减小 60% 到 80%。浏览器接受这些文件之后会自动进行解压缩。

![img](https://3perf.com/static/http-gzip-2-554e0cace5147dd7f9f2ae91b0ddced3-ea61a.png)

基本上，使用 Gzip 已经是生产环境的标准，因此如果你使用一些流行的服务器软件比如 Apache 或者 Nginx，你就可以修改配置文件开启 Gzip 压缩。

[Apache instructions](https://httpd.apache.org/docs/2.4/mod/mod_deflate.html#recommended) · [Nginx instructions](http://nginx.org/en/docs/http/ngx_http_gzip_module.html#example)

__注意：__

使用这些说明启用 Gzip 将会导致服务器动态压缩资源，这会增加服务器响应时间。在大多数情况下你不需要关心这一点，但如果你希望提高响应时间，可以在构建的时候进行资源预压缩。

![img](https://3perf.com/static/http-gzip-3-182a181dab7e6fada817d7b02a90dac2-ea61a.png)

__注意：__

不要对文本文件之外的文件进行 Gzip 压缩！

图像、字体、视频或者其他二进制文件通常已经被压缩过了，因此对它们进行 Gzip 压缩只会延长响应时间。SVG 图片是唯一的例外，因为它也是文本。

##### 三、Brotli 压缩

![img](http://3perf.com/static/http-brotli-1-1f4e48da90b7f031159f9454bd5745ed-ea61a.png)

Gzip 有一个替代品，一种叫 Brotli 的算法。

__Brotli 的优点：__同样的 CPU 载荷下，[它压缩效率比 Gzip 高 20% 到 30%](<https://blogs.akamai.com/2016/02/understanding-brotlis-potential.html>)。就是说可以减少 30% 下载量！

__Brotli 的缺点：__它很年轻，浏览器以及服务器的支持度还不够，所以你不能用它来替代 Gzip。但是可以针对不同的浏览器使用 Gzip 或者 Brotli。

![img](https://3perf.com/static/http-brotli-2-33fdfb713cbd8c74ee252503ee78d1ca-ea61a.png)

Apache 从 2.4.26 开始支持 Brotli，Nginx 有外部模块支持 Brotli。

[Apache instructions](https://httpd.apache.org/docs/trunk/mod/mod_brotli.html#recommended) · [Nginx module](https://github.com/google/ngx_brotli)

__注意：__

不要把 Brotli 的压缩等级设置到最大，那样会让它压缩得比 Gzip 慢。设置为 4 是最好的，[可以让 Brotli 压缩得比 Gzip 更小更快](<https://certsimple.com/blog/nginx-brotli>)。

##### 四、CDN

![img](https://3perf.com/static/http-cdn-1bb73509b6076630c22815eac685d1ad.svg)

现在，我们聊聊 CDN。

什么是 CDN？假设你在美国假设了一个应用。如果你的用户来自华沙，他们的请求不得不从波兰发出，一路颠簸来到美国，然后又得回到波兰。这个请求过程将会消耗很多时间：

——网络请求要跨越很长的一段距离

——网络请求要经过很多路由或者类似设备（每个设备都有一段处理时间）

如果用户想要获取 app 数据，而且只有美国的服务器知道如何处理数据，那上面这些过程好像都是必要的。但对于静态内容而言，上面的请求过程完全没有必要，因为它们请求的只是一些静态内容，完全可以部署到任何服务器上。

![img](https://3perf.com/static/http-cdn-2-402758be60c8e034925636cbdf6e25e7-ea61a.png)

CDN 服务就是用来解决这个问题的。CDN 代表“Content Delivery Network（静态内容分发）”，CDN 服务在全世界提供许多服务器来 “serve” 静态文件。如果要使用的话，只需要在一个 CDN 服务注册，然后上传你的静态文件，然后更新 app 中引用的文件的地址，然后每个用户都会引用离他们最近的服务器上的静态文件了。

根据我们的经验，CDN 基本上能把每个请求的延迟从上百毫秒减少到 5-10 毫秒。考虑到当页面打开时有很多资源要加载，CDN 的优化效果是很惊人的。

##### 五、资源预加载

![img](https://3perf.com/static/http-preload-1-d82205da957c33b18e1dc6379060f8fd-ea61a.png)

你知道吗？谷歌在你开始点击搜索之前已经在加载搜索结果的第一项了。这是因为[三分之一的用户会首先点击第一个搜索结果](<https://searchenginewatch.com/sew/study/2276184/no-1-position-in-google-gets-33-of-search-traffic-study>)，预加载内容可以让用户更快的看到目标页面。

如果你确定你的页面或者资源会在不久之后被用到，浏览器允许你进行预加载。

![img](https://3perf.com/static/http-preload-2-45f6e0e839103daba18eda157a69065a-ea61a.png)

有五种方法可以实现预加载，它们每一种的适用场景都不同：

——`<link rel="dns-prefetch">` 提示浏览器对一个 IP 地址提前进行 DNS 请求。这对于 CDN 请求很有用，此外一些你知道域名但是不知道具体地址的资源的预加载也可以使用。

——`<link rel="preconnect">` 提示浏览器提前连接到某台服务器。在 `dns-prefetch` 适用的场景同样适用，但它可以建立完整的连接并节约很多时间。缺点是打开新的连接很消耗资源，因此不要过度使用。

——`<link rel="prefetch">` 会在后台对资源进行低优先级预加载然后缓存，这个比较有用，比如在进入 SPA 的下一个页面之前加载 bundle。

——`<link rel="preload">` 会在后台对资源进行高优先级的预加载。这对于加载短时间内即将用到的资源而言比较有用。

——`<link rel="prerender">` 会在后台预加载特定页面，然后在不可见的 tab 中渲染页面。当用户进入这个页面时，页面可以立马呈现在用户面前。这是谷歌用于预加载第一条搜索结果的方案。

__注意：__

不要过度使用预加载，虽然预加载能够提升用户体验、加速应用，但是会导致不必要的流量消耗；尤其是在移动端，用户会消耗过多的不要的流量，这同样会降低用户体验。

阅读更多：[Preload, prefetch and priorities in Chrome](https://medium.com/reloading/preload-prefetch-and-priorities-in-chrome-776165961bbf) · [Prefetching, preloading, prebrowsing](https://css-tricks.com/prefetching-preloading-prebrowsing/)

##### 六、总结

![img](https://3perf.com/static/http-summing-up-082232489538895834821c80e8485557-ea61a.png)

HTTP 优化方式: 

—— [压缩 HTML 代码，就像其它资源那样](https://3perf.com/talks/web-perf-101/#http-minify) 

—— 使用 [Gzip](https://3perf.com/talks/web-perf-101/#http-gzip-1) and [Brotli](https://3perf.com/talks/web-perf-101/#http-brotli-1) 压缩文本资源

—— 使用 CDN 节省[静态资源的下载时间](https://3perf.com/talks/web-perf-101/#http-cdn-1) 

—— 预加载[一会将要用到的资源](https://3perf.com/talks/web-perf-101/#http-preload-1)

#### 图片

![img](https://3perf.com/static/images-and-fonts-header-50499af03396cb2ddb836c58f7ef233a-43ccb.png)

继续，说说图片优化。

##### 一、合适的格式

![img](https://3perf.com/static/images-and-fonts-format-b6c822b4de4ef9dd72e9487c448fc021-ea61a.png)

图片消耗了大量的流量，但庆幸的是图片加载不阻塞渲染。但图片优化仍然是必要的，我们需要让图片加载更快、消耗更少的流量。

第一，也是最重要的一点，选择合适的图片格式。

最常见的图片格式是：`svg`、`jpg`、`png`、`webp`和 `gif`。

![img](http://3perf.com/static/images-and-fonts-svg-f4376032ce72be026d1b588044b2bccd-ea61a.png)

`svg` 最适合矢量图，比如 icon 和 logo。

![img](http://3perf.com/static/images-and-fonts-jpg-42a2b638792d59ebfbb499f4bf467de7-ea61a.png)

`jpg` 最适合照片，因为它压缩图片时质量损耗最小，以至于肉眼难以发现。

![img](http://3perf.com/static/images-and-fonts-png-1d90b4845e5a6acf15dc76818829a728-ea61a.png)

`png` 适合没有任何质量损失的光栅图形 - 例如光栅图标或像素艺术。

![img](https://3perf.com/static/images-and-fonts-webp-1-5949f35ed7cafab81d40b49449c68b3a-ea61a.png)

`webp` 最适合照片或者光栅图片，因为它支持有损或者无损压缩。它的压缩比也比 `jpg` 和 `png` 更优秀。

不幸的是 `webp` 只能在 chrome 使用，但是你仍然可以使用 `jpg` 和 `png` 来实现一个 fallback。

![img](https://3perf.com/static/images-and-fonts-webp-2-aa8794a0b9bb47aa9915641ba5bbedbd-ea61a.png)

上面就是具体实现。

这样写的话，支持 `webp` 的浏览器会加载 `webp` 格式的图片，不支持 `webp` 格式的浏览器会加载 `jpg` 最为备用方案。

![img](https://3perf.com/static/images-and-fonts-gif-621eec56683e337fdeb6f4c4b5364e43-ea61a.png)

最后是 `gif`。

不要使用 `gif`，它非常笨重。超过 1M 的 gif 最好使用视频文件代替，可以更好的压缩内容。

See also: [Replace Animated GIFs with Video](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/replace-animated-gifs-with-video/) at WebFundamentals

##### 二、图片压缩

![img](https://3perf.com/static/images-and-fonts-compress-dc3c1b58c7b1dba6478fb1a596f9b902-ea61a.png)

除了使用合适的图片格式以外，图片压缩也可以是优化方案。下面是几种图片压缩方式：

![img](https://3perf.com/static/images-and-fonts-compress-svg-jpg-507266e4d3157fcac157bd358156eab6-ea61a.png)

首先是 `svg`：

——压缩。因为 svg 图片是文本，所以可以移除空格和注释

——简化 path，如果 svg 是自动工具生成的，其内部的 path 可能会很复杂，这种情况下，移除那些不影响 svg 样式的 path

——简化 svg 文件结构，如果 svg 是自动工具生成的，通常会包含很多多余的 meta 元素，移除它们可以减小文件体积

这些优化方式都可以直接使用 [`svgo`](https://github.com/svg/svgo) 实现，它还有 UI 界面：[a great UI for `svgo`](https://jakearchibald.github.io/svgomg/)

![img](https://3perf.com/static/images-and-fonts-compress-svg-jpg-exclamation-4463aaeab9b88ce0264a4b34adf0164f-ea61a.png)

第二个：`jpg`。

——减小图片维度。根据我的经验，这是一个开发人员使用 jpg 常犯的错误

![img](https://3perf.com/static/images-and-fonts-compress-jpg-dimensions-fb07b6833f902f8a8649f78618aa324f-ea61a.png)

这种情况常发生于我们把一张大尺寸的图片塞进一个小尺寸的容器中时。比如我们把一张 2560 * 1440 px 的图片放到一个 533 * 300 px 的容器中。

当这种情况发生时，浏览器会加载过大的文件，然后还要花时间缩小图片，知道能够塞进去那个小小的容器，这些都是无用功。

要解决这个问题，可以直接在你的 PS 或者其他工具中对图片进行编辑；或者你也可以使用 webpack loader（比如 [`responsive-loader`](https://github.com/herrstucki/responsive-loader)）。如果要使用大尺寸图片适配高分屏，可以通过 [`<picture>` 或者 `<img srcset>`](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images) 代替。

![img](https://3perf.com/static/images-and-fonts-compress-jpg-size-1-3d67aff4273363950d51855191804973-ea61a.png)

还可以对 jpg 进行图片降维压缩，图片质量压缩到原来的 70 ～ 80，图片压缩导致的质量损失会很难发现。

![img](https://3perf.com/static/images-and-fonts-compress-jpg-size-2-cc70eb30646fe963643fdc152d0045d2-ea61a.png)

![img](https://3perf.com/static/images-and-fonts-compress-jpg-size-3-b2ff56215fd98c6e3e09a2ba675d54e1-ea61a.png)

上面可以看出压缩后图片质量损失不大。

![img](https://3perf.com/static/images-and-fonts-compress-jpg-size-4-b3927c0b2d88966db1822f267a8dda08-ea61a.png)

但是我们可以看到图片的大小减小了很多。这就是为什么推荐对 jpg 图片进行 70-80 水平的压缩，因为图片信息损失很小，但是体积压缩很大。

![img](https://3perf.com/static/images-and-fonts-compress-jpg-progressive-1-c69d6b1d901946e502df62ed4107e0b6-ea61a.png)

除了以上方式外，我们还可以使用渐进式图片。

![img](https://3perf.com/static/non-progressive-d7466f9732b032f93c6a1c1d2bad6cea.gif)

上方是非渐进式图片加载的方法。

![img](https://3perf.com/static/progressive-5e991a649f4184c1e7e5206f7c0f4167.gif)

这是一张渐进式的图片的加载方式。

可以通过 PS 或者 Gimp 制作渐进式图片。也可以使用 webpack-loader（比如 [`image-webpack-loader`](https://www.npmjs.com/package/image-webpack-loader)）或者其他工具。

__注意：__

渐进式图片可能比常规图片更大，而且解码更慢。

![img](https://3perf.com/static/images-and-fonts-compress-png-b421fc0ede9890d466728caeea78f10f-ea61a.png)

第三，`png`。

——使用隔行扫描 PNG。 隔行扫描 PNG 的工作方式与渐进式 JPEG 相同：它从低质量开始渲染，但在加载时进行改进。 但它不是适合所有场景。例如，逐步加载 PNG 图标看起来很奇怪 - 但它可能适用于其他某些图像。

——使用索引颜色。 通过使用索引颜色，PNG 图片将其所有颜色放入调色板中并使用它来引用每种颜色。 这使得每个像素所需的字节数更小，并且可能有助于降低整体图像权重。 由于调色板大小有限（最多256种颜色），因此此解决方案不适用于具有大量颜色的图像。

这两种方式都可以通过图片编辑器或者 [`image-webpack-loader`](https://www.npmjs.com/package/image-webpack-loader) 或者其他工具实现。

![img](https://3perf.com/static/images-and-fonts-compress-tools-33c148f4344a996e9b77924fe894d1b4-ea61a.png)

以上的所有优化都可以使用自动化工具完成，之前都已经提到过，但是这里再总结一下：

— webpack has [`image-webpack-loader`](https://www.npmjs.com/package/image-webpack-loader) which runs on every build and does pretty much every optimization from above. Its default settings are OK

— For you need to optimize an image once and forever, there’re apps like [ImageOptim](https://imageoptim.com/mac) and sites like [TinyPNG](http://tinypng.com/).

— If you can’t plug a webpack loader into your build process, there’re also a number of CDNs and services that host and optimize images for you (e.g., [Akamai](https://www.akamai.com/), [Cloudinary](https://cloudinary.com/), or [imgix](https://www.imgix.com/)).

##### 三、总结

![img](https://3perf.com/static/images-summing-up-69d3f65568e3944d59b32693479981a9-ea61a.png)

图片优化总结：

——[选择合适的图片格式](https://3perf.com/talks/web-perf-101/#images-format) 

——通过图片降维、质量压缩或者使用渐进式图片[优化图片加载时间](https://3perf.com/talks/web-perf-101/#images-compress)

#### 字体

![img](https://3perf.com/static/fonts-header-87b940d0590df28ac814bb85961eec40-43ccb.png)

最后一个优化方式就是字体了。

有时候页面加载好了，所有的样式、布局都已经可见了，但是字体还没有出现或者显示异常，这就是字体问题所导致的，自定义字体尚未下载完毕，这个时候浏览器会等待几秒，如果仍然未下载，浏览器才会使用备用字体作为替代。

这种行为在某种程度上避免了字体的闪烁，但是在缓慢的网络条件下，这种行为使得页面加载变得缓慢。

##### 一、指定 fallback 字体

我们需要了解一下如何优化这种情况。

![img](https://3perf.com/static/fonts-fallback-1-d8401282abc0748360eaeacb0db5f4b3-ea61a.png)

首先，要记得设置 fallback 字体。

fallback 字体会在自定义字体无法下载或者下载时间过长时被使用。它在 CSS 的 `font` 或者 `font-family` 的第一个指定字体后面指定，比如上方的`Arial, sans-serif`。

fallback 字体应当是比较流行的[内置字体](<https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Fundamentals#Web_safe_fonts>)（比如 Georgia）；也可以是比较通用的字体系列（如 serif 或者 sans-serif）；通常情况下，即使你指定了内置的字体作为 fallback，但是你仍然需要添加一个通用的字体系列——因为内置字体可能也会在某些设备上缺失。

![img](https://3perf.com/static/fonts-fallback-2-396a4fe852fc760e157adc8277bd2b12-ea61a.png)

没有 fallback 字体的话，一旦自定义字体缺失，浏览器会使用默认的 [serif font](https://en.wikipedia.org/wiki/Serif) 进行渲染。这样可能会导致页面比较难看。

![img](https://3perf.com/static/fonts-fallback-3-99b86dda1c66097ac20eb72f3c062612-ea61a.png)

使用 fallback 字体，至少你有机会定义一个和你的自定义字体相近的字体作为备用方案。

##### 二、使用 `font-display`

![img](https://3perf.com/static/fonts-font-display-2-4cdd79d523085b42a08e2d2c8ce0b76c-ea61a.png)

第二点优化，使用 CSS 的 `font-display` 属性指定自定义字体。

`font-display` 属性会调整自定义字体的应用方式。默认情况下，它会设置为 `auto`，在大部分主流浏览器中，意味着浏览器会等待自定义字体加载 3s。这意味着如果网络太慢的话，用户需要等待 3s 后字体才会显示。

这很不好，为了优化这一点，指定 `font-display`。

Note: in Microsoft Edge, the `font-display: auto` behavior is different. With it, if the custom font is not cached, Edge immediately renders the text in the fallback font and substitutes the custom font later when it’s loaded. This is not a bug because `font-display: auto` lets browsers define the loading strategy.

![img](https://3perf.com/static/fonts-font-display-3-2acef56eedbe72cf47d924315c689000-ea61a.png)

有两个 `font-display` 的值我认为比较适用于大部分情况。

第一个是 `font-display: fallback`。这样指定的话，浏览器会使用最早能够获得的字体立即渲染，不管是已经缓存的自定义字体还是 fallback 字体。如果自定义字体没有被缓存的话，浏览器会下载它。如果下载得足够快（通常是 3s 内），浏览器会使用自定义字体替换 fallback 字体。

这种情况下，用户可能会在读 fallback 字体的文本时，浏览器突然进行字体替换，这对于用户体验而言并不是很差，总比不显示任何字体要强。

![img](https://3perf.com/static/fonts-font-display-4-d7882d2cab46bacb137617fc34d8ef2c-ea61a.png)

第二个适用的 `font-display` 值是 `optional`。使用这个值，浏览器同样会立即使用可获得的字体进行文本渲染：不管是已缓存的自定义字体还是 fallback 字体。但是当自定义字体未缓存时，在下载好自定义字体后，浏览器不会立即替换已有的 fallback 字体，直到页面下一次刷新。

这种行为意味着用户始终只会看到一种字体，不会出现字体替换的情况。

![img](https://3perf.com/static/fonts-font-display-5-f87ac4e00b54132b60d53740013600dc-ea61a.png)

那我们该如何选择这两个值呢？

我相信这是一个品味问题。 我个人更喜欢用自定义字体展示文本，因此我选择 `font-display：fallback` 值。 如果你觉得访问者第一次访问时看到 fallback 字体的页面没有什么关系，那么 `font-display：optional` 对您来说非常有用。

Note: this `font-display` trick is not applicable to icon fonts. In icon fonts, each icon is usually encoded by a rarely used Unicode character. Using `font-display` to render icons with a fallback font will make random characters appear in their place.

##### 三、总结

![img](https://3perf.com/static/fonts-summing-up-4b5fbd603c3ab193870f27cb9135f797-ea61a.png)

字体优化方案的总结：

—— [指定合适的 fallback（备用）字体](https://3perf.com/talks/web-perf-101/#fonts-fallback-1) (还有通用的字体系列) 

—— [使用 `font-display`](https://3perf.com/talks/web-perf-101/#fonts-font-display-1) 来配置自定义字体的应用方式。

### 有哪些可用的优化工具

![img](https://3perf.com/static/tools-header-8651e37f828542d575a7de00a7ef0738-43ccb.png)

最后是一些有助于页面性能优化的工具。

![img](https://3perf.com/static/tools-pagespeed-insights-353b54d55a509871e22ba87f4cb7d213-ea61a.png)

第一个是 [Google PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)。

![img](https://3perf.com/static/tools-lighthouse-1847d59a498b116b741c005e7a2377a2-ea61a.png)

第二个是 [Lighthouse](https://developers.google.com/web/tools/lighthouse/)。

![img](https://3perf.com/static/tools-webpagetest-34dcba654fce18f956214bcc8f9c94d5-ea61a.png)

第三个是 [WebPageTest](https://webpagetest.org/)。

最后一个是 webpack 插件：[webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)。

具体的介绍就没写了，点进去直接用就知道啦。

感谢阅读！

原作者推特：[@iamakulov](https://twitter.com/iamakulov)。

Thanks to [Arun](https://twitter.com/arunbasillal), [Anton Korzunov](https://twitter.com/theKashey), [Matthew Holloway](https://twitter.com/hollowaynz), [Bradley Few](https://twitter.com/bradleyfew), [Brian Rosamilia](https://twitter.com/BrianRosamilia),[Rafael Keramidas](https://twitter.com/iamkeraf), [Viktor Karpov](https://twitter.com/vitkarpov), and Artem Miroshnyk (in no particular order) for providing feedback on drafts.

水平有限，难免存在纰漏，敬请大家斧正！