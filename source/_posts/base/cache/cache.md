---
title: 缓存系列 - 强缓存和协商缓存
categories: base
tags: #文章標籤 可以省略
  - cache
---

### 强缓存

**原理**

当浏览器对某个资源的请求命中了强缓存时，返回的 HTTP 状态为 200，在 chrome 的开发者工具的 network 里面 size 会显示为 from cache，强缓存是利用 Expires 或者 Cache-Control 这两个 HTTP Response Header 实现的，它们都用来表示资源在客户端缓存的有效期。

如果 Cache-Control 与 Expires 同时存在的话，Cache-Control 的优先级高于 Expires。

**配置**

- 通过代码的方式，在 Web 服务器返回的响应 Header 中添加 Expires 和 Cache-Control
- 通过配置 Web 服务器的方式，让 Web 服务器在响应资源的 Header 统一添加 Expires 和 Cache-Control

**总结**

针对静态资源使用，动态资源需要慎用

### 协商缓存

**原理**

浏览器对某个资源的请求没有命中强缓存，就会发一个请求到服务器，验证协商缓存是否命中，如果协商缓存命中，请求响应返回的 HTTP 状态为 304 并且会显示一个 Not Modified 的字符串。

**配置**

协商缓存是利用的是 [Last-Modified，If-Modified-Since] 和 [ETag、If-None-Match] 这两对 Header 来管理。

Last-Modified && If-Modified-Since

- 1.浏览器第一次向服务器请求一个资源时，服务器在返回这个资源的同时，会在 Respone 的 Header 加上 `Last-Modified`，这个 Header 表示这个资源在服务器上的最后修改时间
- 2.浏览器再次向服务器请求这个资源时，会在 Request 的 Header 上加上 `If-Modified-Since`，这个 Header 的值就是上一次请求时返回的 Last-Modified 的值
- 3.服务器再次收到资源请求时，根据浏览器传过来 `If-Modified-Since` 和资源在服务器上的最后修改时间判断资源是否有变化
  - 3.1.如果没有变化则返回 `304 Not Modified`，但是不会返回资源内容
  - 3.2.如果有变化，就正常返回资源内容
  - 3.3.当服务器返回 `304 Not Modified` 的响应时，Response Header 中不会再添加 `Last-Modified`，因为既然资源没有变化，那么 Last-Modified 也就不会改变
- 4.浏览器收到 304 的响应后，就会从缓存中加载资源
- 5.如果协商缓存没有命中，浏览器直接从服务器加载资源，Header 中的 `Last-Modified` 在重新加载的时候会被更新，下次请求时，`If-Modified-Since` 会启用上次返回的 Last-Modified 值

这就是 Last-Modified && If-Modified-Since 执行全过程。

<b>存在问题</b>

[Last-Modified，If-Modified-Since] 都是根据服务器的时间来返回 Header，一般来说，在没有调整服务器时间和篡改客户端缓存的情况下，这两个 Header 配合起来管理协商缓存是非常可靠的。但是有时候服务器上资源其实有变化，但是最后修改时间却没有变化的情况，而这种问题又很不容易被定位出来，而当这种情况出现的时候，就会影响协商缓存的可靠性。

ETag && If-None-Match

- 1.浏览器第一次向服务器请求一个资源时，服务器在返回这个资源的同时，在 Respone 的 Header 加上 `ETag`
  - 1.1.这个 Header 是服务器根据当前请求的资源的内容生成一个唯一标识，这个唯一标识是一个字符串，只有资源有变化这个串才会变化，跟最后修改时间没有关系，所以能很好的解决 `Last-Modified` 的问题
- 2.浏览器再次向服务器请求这个资源时，在 Request 的 Header 上加上 `If-None-Match`，这个 Header 的值就是上一次请求时返回的 `ETag` 的值
- 3.服务器再次收到资源请求时，根据浏览器传过来 `If-None-Match` 和根据资源内容生成一个新的 `ETag` 进行对比
  - 3.1.如果这两个值相同就说明资源没有变化，否则就是有变化
  - 3.2.如果没有变化则返回 304 Not Modified，但是不会返回资源内容
  - 3.3.如果有变化，就正常返回资源内容
- 4.浏览器收到 304 的响应后，就会从缓存中加载资源
- 5.与 `Last-Modified` 不一样的是，当服务器返回 `304 Not Modified` 的响应时，由于 ETag 重新生成过，Response Header 中还会把这个 ETag 返回，即使这个 ETag 跟之前的没有变化

这就是 ETag && If-None-Match 执行全过程。

### 强缓存和协商缓存

- 浏览器在加载资源时，先根据这个资源的 HTTP Header 判断它是否命中强缓存，如果命中强缓存，浏览器直接从自己的缓存中读取资源，不会发送请求到服务器。比如某个 CSS 文件，如果浏览器在加载它所在的网页时，这个 CSS 文件的缓存配置命中了强缓存，浏览器就直接从缓存中加载这个 CSS，连请求都不会发送到网页所在服务器
- 当强缓存没有命中的时候，浏览器一定会发送一个请求到服务器，通过服务器端依据资源的 HTTP Header 验证这个资源是否命中协商缓存，如果协商缓存命中，服务器会将这个请求返回，但是不会返回这个资源的内容，而是告诉客户端可以直接从缓存中加载这个资源，于是浏览器就又会从自己的缓存中去加载这个资源
- 强缓存与协商缓存的共同点是：如果命中，都是从客户端缓存中加载资源，而不是从服务器加载资源数据；区别是：强缓存不发请求到服务器，协商缓存会发请求到服务器
- 当协商缓存也没有命中的时候，浏览器直接从服务器加载资源数据

### Cache-Control 流程图

![img](https://github.com/slogeor/images/blob/master/fe/2016/js/cache.06.png?raw=true)

### 协商缓存流程图

![img](https://github.com/slogeor/images/blob/master/fe/2016/js/cache.05.png?raw=true)

### 总结

- 强缓存不发请求到服务器，所以有时候资源更新了浏览器并不知道
- 协商缓存会发请求到服务器，所以资源是否更新，服务器肯定会知道
