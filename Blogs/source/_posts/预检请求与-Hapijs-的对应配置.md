---
title: 预检请求与 Hapijs 的对应配置
date: 2019-01-15 22:49:19
categories:
  - HTTP
  - Hapijs
---

不久前在公司写了一个基于 Hapijs 的后端项目，感觉这个框架很有自己的特点，跟 Express 和 Koa 的区别比较大，体现了配置大于编码的思想。用起来很方便，据说 Walmart 团队用这个框架扛住了黑五的流量，看起来在实际项目中也有可用性，推荐大家尝试一下～

有点跑题了，这篇文章主要写我在开发过程中所遇到的一个问题，以及我从这个问题所学习到的东西，然后我是怎么解决这个问题的。

<!-- more -->

## 一、问题

我的项目需求是写一个 App 版本管理器，前后端都由我开发。前端分为两个部分：运营人员写版本更新说明的内部系统和 App 访问的产品页；后端就是对 App 版本进行管理的 CURD 接口。重点在于三个部分的程序部署在三台服务器上，前端的两个系统在不同的服务器对第三个服务器上的接口进行数据请求，这就不可避免的涉及到了跨域。

当然，只是跨域的话也不难解决，添加 `Access-Control-Allow-Origin` 为要跨域的域名就 OK 了，或者直接赋值为 `*`。但是我的部分接口涉及鉴权，通过 JWT 进行校验，如果 JWT 不合法，那么会返回 401 Unauthorized 错误；而我的 JWT 是通过请求头的自定义字段 `authorization` 带到服务器的，这就导致一个更加麻烦的问题出现了 —— 预检请求。

## 二、收获

### 什么是预检请求？

预检请求（preflight request），是一个跨域请求，用来校验当前跨域请求能否被理解。

它使用 HTTP 的 OPTIONS 请求，一般会包括一下请求头：`Access-Control-Request-Method`，`Access-Control-Request-Headers` 和 `Origin`。

预检请求通常在必要的时候由浏览器自动发起，不需要程序员进行干预。

如果我们想要知道服务器是否支持一个 `DELETE` 请求，在发送 `DELETE` 请求之前，服务器通常会发送一个如下的预检请求：

```
OPTIONS /resource/foo 
Access-Control-Request-Method: DELETE 
Access-Control-Request-Headers: origin, x-requested-with
Origin: https://foo.bar.org
```

如果服务器允许使用 `DELETE` 方法的话，会返回如下响应头；其中 `Access-Control-Allow-Methods` 会列出 `DELETE` 方法，代表服务器支持这个方法。

```
HTTP/1.1 200 OK
Content-Length: 0
Connection: keep-alive
Access-Control-Allow-Origin: https://foo.bar.org
Access-Control-Allow-Methods: POST, GET, OPTIONS, DELETE
Access-Control-Max-Age: 86400
```

> 以上资料来源于 [MDN](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)

由此可知，预检请求是一个用于校验服务器是否支持当前方法以及是否能够理解当前请求的一种请求，它区别于一般的请求，不由代码发起，而在必要的时候由浏览器自动发出。

所以这里就出问题了，如果我们不知道什么时候浏览器会发出预检请求，那么服务器没有做处理的话就会导致 CORS 报错的出现。

接下来再深入一点。

### 预检请求与普通请求的区别

满足以下条件的请求就是**简单请求**：

* 一、请求方法属于下面三种方法之一：

  * HEAD
  * POST
  * GET

* 二、HTTP 的请求头信息超出一下范围：

  * Accept
  * Accept-Language
  * Content-Language
  * Last-Event-ID
  * Content-Type：超出这三个的范围：

    * application/x-www-form-urlencoded
    * multipart/form-data
    * text/plain

不满足以上条件的请求就是非简单请求。

如果是简单的 CORS 请求，浏览器会自动在请求头中添加一个 Origin 请求头字段，如果响应头对应的 `Access-Control-Allow-Origin` 没有包含 Origin 所指定的域，那么就会报 CORS 错误，请求失败。所以服务器的响应要添加对应的响应头。

如果是非简单的 CORS 请求，那么会有一次预检请求，在正是请求之前发出一个 OPTIONS 请求对服务器进行检测。

除了有 Origin 以外，预检请求的请求头还包括一下两个特殊字段：

* `Access-Control-Request-Method`：表示 CORS 请求要用到的请求方法。

* `Access-Control-Request-Headers`：这是一个用逗号分割的字符串，指出 CORS 请求要附加的请求头。

服务器的响应可以包含以下字段：

* `Access-Control-Allow-Methods`：逗号分割的字符串，表示允许的跨域请求方法。

  比如：
  ```
  Access-Control-Allow-Methods: PUT, POST, GET, OPTIONS
  ```

* `Access-Control-Allow-Headers`：如果浏览器请求包含 `Access-Control-Request-Headers` 字段，那么服务器中该响应头也是必须的，也是一个由逗号分隔的字符串，表示服务器支持的请求头。

  比如：

  ```
  Access-Control-Allow-Headers: authorization
  ```

* `Access-Control-Max-Age`：可选字段，设置当前预检请求的有效期，单位为秒。

* `Access-Control-Allow-Credentials`：可选字段。默认情况下，CORS 请求不携带 cookie，如果服务器想要 cookie，需要指定该请求头为 `true`。

## 三、解决方法

* 避免出现预检请求，需要使得你的请求满足简单请求的两个条件。
  
  比如在使用 JWT 鉴权时，可能会把你的 token 放在请求头的 authorization 字段，因为这个字段超出了简单请求的范围，所以请求会变成非简单请求。这时可以不把 token 放在 authorization 请求头中。

* 出现预检请求后，进行服务器配置，分别设置好 `Access-Control-Allow-Origin`、`Access-Control-Allow-Methods` 和 `Access-Control-Allow-Headers`，使得你的非简单请求能够通过预检请求。

* 如果使用 Hapijs 的话，只需要在路由配置中增加 `cors: true` 配置即可。

## 参考

* [MDN](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)
* [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
* [cors跨域之简单请求与预检请求（发送请求头带令牌token）](https://segmentfault.com/a/1190000009971254)