---
title: 浅谈 JSONP
date: 2019-04-13 17:54:06
categories:
  - JavaScript
---

说起跨域的解决方案，总是会说到 JSONP，但是很多时候都没有仔细去了解过 JSONP，可能是因为现在 JSONP 用的不是很多（多数时候都是配置响应头实现跨域），也可能是因为用 JSONP 的场景一般都是用 jQuery 来实现，所以对 JSONP 知之甚少。

JSONP 的本意是 JSON with Padding，即填充式 JSON。为什么叫填充式呢？因为服务端不会直接返回 JSON 格式数据给客户端，它会拼接成一个字符串，这个字符串被拿到客户端执行。这是对于 JSON 的一种应用。

### JSONP 的原理是什么？

发明 JSONP 的老头子们发现虽然[同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)（CORS）限制了 ajax 对于其它服务器的访问，但是并不能限制 HTML 的资源请求。

<!-- more -->

比如在 HTML 中，img、script、link 等标签完全可以访问任何地址的资源。而其中的 script 标签为跨域请求提供了一种新的思路，因为 script 请求的是一段可执行的 JavaScript 代码。我们可以把之前直接从服务器返回的数据封装到 JavaScript 代码中，然后在前端再使用这些数据。这就是 JSONP 的实现原理。

### 为什么 JSONP 只能使用 GET 方法？

使用 jQuery 的 `$.ajax` 进行 JSONP 请求时，type 属性总是选择为 `GET`，如果填为 `POST`，就会报错，这是为什么呢？其实理解 JSONP 的原理之后，这就很好理解了。原因就是 script 标签的资源请求只能是 `GET` 类型，目前为止我还没有见过 POST 类型的资源请求～

### 如何实现 JSONP？

前端小白不理解 JSONP 的另一个原因就在于 JSONP 不只是前端这一块的任务，只靠前端是无法实现的，后端也必须做相应处理。

前端请求一个专用的接口获取数据，请求的数据会以 JavaScript 代码的形式返回，假如数据如下：

```json
{
  "name": "russ",
  "age": 20
}
```

那要构造成什么样的 JavaScript 代码才能被前端使用到呢？最容易想到的就是把数据赋值给一个全局变量，或者把数据扔到函数里面。扔给全局变量的话会导致一些问题（全局暴露、命名冲突、数据处理逻辑分散……），所以把数据扔给一个专门处理数据的函数比较合适。这也是 JSONP 所采用的方案。所以拼接出来的字符串（也是后端返回的 js 代码）基本如下：

```js
callback({
  "name": "russ",
  "age": 20
})
```

那么前端就需要在 script 请求返回之前定义好 callback 这个函数，以便在 script 返回之后可以顺利加载执行。这里需要前后端约定好回调函数的名称。当然可以前端传递回调的名称给后端，后端根据前端传递的名称进行 JavaScript 代码拼接，jQuery.ajax 就有这种实现,允许前端自定义回调的名称。

先讲讲后端实现，因为后端实现起来比较简单～

#### 后端处理

> 以下实现均使用 Nodejs。

Nodejs 实现代码如下：

```js
const http = require('http');
const { parse } = require('url');

// 假设这是在数据库中找到的数据~
const data = {
  name: 'russ',
  age: 20,
  gender: 'male'
};

const server = http.createServer((req, res) => {
  const url = parse(req.url, true); // 解析 url

  // 只有路由为 `/user` 的 GET 请求会被响应
  if (req.method === 'GET' && url.pathname === '/user') {
    const { callback } = url.query; // 获取 callback 回调名称
    
    if (callback) // 如果传递了 callback 参数，说明是 JSONP 请求
      return res.end(`${callback}(${JSON.stringify(data)})`);
    else // 没传递 callback，直接当做不跨域的 GET 请求返回数据
      return res.end(JSON.stringify(data));
  }
  return res.end('Not Found'); // 不匹配的路由返回错误
});

server.listen(3000, () => {
  console.log('Server listening at port 3000...');
});
```

可以直接在浏览器中请求查看结果～

#### 前端处理

伴随着很多功能强大的 api 的出现，我们在很多场景下都可以直接弃用 jQuery（参考[nefe/You-Dont-Need-jQuery 
](https://github.com/nefe/You-Dont-Need-jQuery)）。而如果我们的页面没有使用 jQuery 的时候，我们就需要手动实现 JSONP 了～

前面说过 JSONP 的原理是 script 标签的资源请求，所以前端的处理就是构造 script 标签发起请求。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>JSONP</title>
</head>
<body>
  <!-- 点击的时候调用 fetchJSON -->
  <button onclick="fetchJSON()">Fetch</button>
</body>
<script>
  // 定义了 fetchJSON 函数。
  function fetchJSON() {
    // 内部调用 jsonp 函数实现接口的 jsonp 访问。
    jsonp('http://localhost:3000/user?').then(data => {
      console.log(data);
    }).catch(err => {
      console.log(err);
    })
  }
  function jsonp(url) {
    let $script = document.createElement('script'), // 先构造一个 script 元素
      callbackName = `callback_${Date.now()}`; // 先定义回调名称，加时间戳防止缓存

    // 返回 promise 对象，方便后续处理
    return new Promise((resolve, reject) => {
      // 在发起请求之前，先定义好回调函数
      window[callbackName] = (res) => {
        // 请求结束之后清除全局变量
        window[callbackName] = undefined;
        // 移除之前挂载的 script 元素
        document.head.removeChild($script);
        // 清空 $script
        $script = undefined;
        resolve(res);
      };
      // 绑定 src，及请求地址
      $script.src = `${url}callback=${callbackName}`;
      // 绑定 error 处理函数
      $script.onerror = err => {
        reject(err);
      };
      // 挂在 script 元素到 head，此时才开始发起请求～
      document.head.appendChild($script); // 开始请求
    });
  }
</script>
</html>
```

当然上面的代码没有做兼容性处理，在低级浏览器使用时需要做一下处理，但是其他原理是一样的。

### JSONP 有什么缺点呢？

大致有两点：

一、安全性问题

JSONP 会从其它域加载 JavaScript 脚本并直接执行，如果 JavaScript 脚本中包含恶意攻击代码，那我们的网站将会受到威胁。所以当我们访问非自己维护的服务器的 JSONP 接口时，需要留心。

二、错误处理

script 标签的 onerror 函数在 HTML5 才定义，并且即使我们定义了 onerror 处理函数，我们也不容易捕捉到错误发生的原因。所以这也是一大缺点，至于具体表现可以单独运行上面的前端代码试试，看看错误发生时（后端服务未启动），前端控制台打印出来的错误对象是什么样的。