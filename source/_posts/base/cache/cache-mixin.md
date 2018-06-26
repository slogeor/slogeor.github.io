---
title: HTTP 缓存 -- 大杂烩
categories: base
tags: #文章標籤 可以省略
  - cache
---

### 浏览器缓存

浏览器缓存是将文件保存在客户端，在同一个会话过程中会检查缓存的副本是否足够新，在后退网页时，访问过的资源可以从浏览器缓存中拿出使用。

通过减少服务器处理请求的数量，用户将获得更快的体验。

### Expires

`HTTP 1.0` 提出的缓存过期时间，用来指定资源到期的时间，是服务器端的具体的时间点，是一个绝对时间。

Expires = max-age + 请求时间，需要和 `Last-modified` 结合使用。

**过程**

浏览器第一次向服务器请求一个资源，服务器在返回这个资源的同时，在 respone 的 Header 加上 Expires 的 Header，如：

![img](https://github.com/slogeor/images/blob/master/fe/2016/js/cache.01.png?raw=true)

- 浏览器在接收到这个资源后，会把这个资源连同所有 Response Header 一起缓存下来
- 浏览器再请求这个资源时，先从缓存中寻找，找到这个资源后，拿出它的 Expires 跟当前的请求时间比较，如果请求时间在 Expires 指定的时间之前，就能命中缓存，否则没有命中
- 如果缓存没有命中，浏览器直接从服务器加载资源时，Expires Header 在重新加载的时候会被更新

**缺点**

Expires 是较老的强缓存管理 Header，由于它是服务器返回的一个绝对时间，在服务器时间与客户端时间相差较大时，缓存管理容易出现问题，比如随意修改下客户端时间，就能影响缓存命中的结果。

### Cache-Control

`HTTP 1.1` 的时候，提出了一个新的 Header，就是 Cache-Control ，这是一个相对时间，在配置缓存的时候，以秒为单位，用数值表示。

**过程**

浏览器第一次向服务器请求一个资源，服务器在返回这个资源的同时，在 Respone 的 Header 加上 Cache-Control 的 Header，如：

![img2](https://github.com/slogeor/images/blob/master/fe/2016/js/cache.01.png?raw=true)

- 浏览器在接收到这个资源后，会把这个资源连同所有 Response Header 一起缓存下来
- 浏览器再请求这个资源时，先从缓存中寻找，找到这个资源后，根据它第一次的请求时间和 Cache-Control 设定的有效期，计算出一个资源过期时间，再拿这个过期时间跟当前的请求时间比较，如果请求时间在过期时间之前，就能命中缓存，否则没有命中
- 如果缓存没有命中，浏览器直接从服务器加载资源时，Cache-Control Header 在重新加载的时候会被更新

Cache-Control 描述的是一个相对时间，在进行缓存命中的时候，都是利用客户端时间进行判断，所以相比较 Expires，Cache-Control 的缓存管理更有效，安全一些。

**属性**

1.max-age


max-age（单位为s）设置缓存最大的有效时间，定义的是时间长短。

比如 slogeor.com 上的 CSS 资源，max-age=2592000，也就是说缓存有效期为 2592000 秒（也就是30天）。于是在 30 天内都会使用这个版本的资源，即使服务器上的资源发生了变化，浏览器也不会得到通知。max-age 会覆盖掉 Expires。

2.s-maxage

s-maxage（单位为s）同 max-age，只用于共享缓存（比如CDN缓存）

比如，当 s-maxage=60 时，在这 60 秒中，即使更新了 CDN 的内容，浏览器也不会进行请求。也就是说 max-age 用于普通缓存，而 s-maxage 用于代理缓存。如果存在 s-maxage，则会覆盖掉 max-age 和 Expires Header。

3.public

指定响应会被缓存，并且在多用户间共享，默认为 public。

4.private

响应只作为私有的缓存，不能在用户间共享。如果要求 HTTP 认证，响应会自动设置为 private。

5.no-cache

no-cache 指定不缓存响应，表明资源不进行缓存。

设置了 no-cache 之后并不代表浏览器不缓存，而是在缓存前要向服务器确认资源是否被更改。因此有的时候只设置 no-cache 防止缓存还是不够保险，还可以加上 private 指令，将过期时间设为过去的时间。

6.no-store

no-store 绝对禁止缓存，一看就知道如果用了这个命令当然就是不会进行缓存啦～每次请求资源都要从服务器重新获取。


### Last-modified VS Etag

**Last-modified**

服务器端文件的最后修改时间，需要和 cache-control 共同使用，是检查服务器端资源是否更新的一种方式。当浏览器再次进行请求时，会向服务器传送 If-Modified-Since 报头，询问 Last-Modified 时间点之后资源是否被修改过。如果没有修改，则返回码为 304，使用缓存；如果修改过，则再次去服务器请求资源，返回码和首次请求相同为 200，资源为服务器最新资源。

**ETag**

根据实体内容生成一段 hash 字符串，标识资源的状态，由服务端产生。浏览器会将这串字符串传回服务器，验证资源是否已经修改。

使用 ETag 可以解决 Last-modified 存在的一些问题。

某些服务器不能精确得到资源的最后修改时间，这样就无法通过最后修改时间判断资源是否更新
如果资源修改非常频繁，在秒以下的时间内进行修改，而 Last-modified 只能精确到秒
一些资源的最后修改时间改变了，但是内容没改变，使用ETag就认为资源还是没有修改的
缓存流程

### 浏览器刷新行为

**刷新**

点击刷新按钮或者按 F5，会触发这种行为

浏览器直接对本地的缓存文件过期，但是会带上If-Modifed-Since，If-None-Match（如果上一次 Response 带 Last-Modified, Etag）这就意味着服务器会对文件检查新鲜度，返回结果可能是 304，也有可能是 200。

**强制刷新**

用户按 Ctrl+F5

浏览器不仅会对本地文件过期，而且不会带上 If-Modifed-Since，If-None-Match，相当于之前从来没有请求过，返回结果是 200。

**地址栏回车**

浏览器发起请求，按照正常流程，本地检查是否过期，然后服务器检查新鲜度，最后返回内容。

**点击**

和回车的效果是一样的。

**总结**

用户主动触发的页面刷新行为（比如刷新按钮、右键刷新、F5 等），会导致浏览器放弃本地缓存，使用协商缓存（ 304 缓存）。


### 200 VS 304

- 200 from memory cache 不访问服务器，直接读缓存，从内存中读取缓存。此时的数据时缓存到内存中的，当kill进程后，数据将不存在
- 200 from disk cache 不访问服务器，直接读缓存，从磁盘中读取缓存，当kill进程时，数据还是存在
- 304 Not Modified 访问服务器，发现数据没有更新，服务器返回此状态码。然后从缓存中读取数据

### 参考链接

- [http://www.cnblogs.com/lyzg/p/5125934.html](http://www.cnblogs.com/lyzg/p/5125934.html)
- [http://www.alloyteam.com/2016/03/discussion-on-web-caching/](http://www.alloyteam.com/2016/03/discussion-on-web-caching/)
