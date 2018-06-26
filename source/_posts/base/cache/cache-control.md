---
title: HTTP 缓存 -- Cache-Control
categories: base
tags: #文章標籤 可以省略
  - cache
---

### HTTP 头信息

![img1](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/images/http-request.png?hl=zh-cn)


服务器返回一个 1024 字节的响应，指示客户端将其缓存最多 120 秒，并提供一个验证令牌`x234dff`，可在响应过期后用来检查资源是否被修改。


### Etag

- 服务器使用 ETag HTTP 标头传递验证令牌
- 验证令牌可实现高效的资源更新检查：资源未发生变化时不会传送任何数据


ETag 旨在解决的问题: 服务器生成并返回的随机令牌通常是文件内容的哈希值或某个其他指纹。客户端不需要了解指纹是如何生成的，只需在下一次请求时将其发送至服务器。如果指纹仍然相同，则表示资源未发生变化，您就可以跳过下载。

![img2](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/images/http-cache-control.png?hl=zh-cn)

客户端自动在 `If-None-Match` HTTP 请求标头内提供 ETag 令牌。服务器根据当前资源核对令牌。如果它未发生变化，服务器将返回 `304 Not Modified` 响应，告知浏览器缓存中的响应未发生变化，可以再延用 120 秒。


### Cache-Control

- 每个资源都可通过 Cache-Control HTTP 标头定义其缓存策略
- Cache-Control 指令控制谁在什么条件下可以缓存响应以及可以缓存多久


**no-cache**

no-cache 表示必须先与服务器确认返回的响应是否发生了变化，然后才能使用该响应来满足后续对同一网址的请求。

no-cache 会发起往返通信来验证缓存的响应，但如果资源未发生变化，则可避免下载。

**no-store**

直接禁止浏览器以及所有中间缓存存储任何版本的返回响应。

**public**

非必须。

**private**

通常只为单个用户缓存，因此不允许任何中间缓存对其进行缓存。

**max-age**

指令指定从请求的时间开始，允许获取的响应被重用的最长时间。

### Cache-Control 策略图

![img](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/images/http-cache-decision-tree.png?hl=zh-cn)


### 检查缓存清单

- 使用一致的网址
- 确保服务器提供验证令牌 (ETag)
- 确定中间缓存可以缓存哪些资源
- 为每个资源确定最佳缓存周期
- 确定最适合您的网站的缓存层次结构
- 最大限度减少搅动

### 参考链接
- [HTTP 缓存
](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=zh-cn#cache-control)
