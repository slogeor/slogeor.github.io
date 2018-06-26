---
title: 性能优化－预加载
categories: base
tags: #文章標籤 可以省略
  - performance
---

### 概念

预加载是浏览器对将来可能被使用资源的一种暗示，一些资源可以在当前页面使用到，一些资源可能在将来的某些页面中被使用。作为开发人员，我们比浏览器更加了解我们的应用，所以我们可以对核心资源使用预加载。

### 为何需要预加载

- 用户可能是第一次访问网站，此时还无缓存
- 用户可能清空了缓存
- 缓存可能已经过期，资源将重新加载
- 用户访问的缓存文件可能不是最新的，需要重新加载

### 预加载技术

**1、DNS预解析dns-prefetch**

通过DNS预解析来告诉浏览器未来我们可能从某个特定的URL获取资源，当浏览器真正使用到该域名下的某个资源时就可以尽快地完成DNS解析

如果将来可能从`example.com`获取图片或视频资源，那么可以在文档顶部的`<head>`标签中加入以下内容

`<link rel="dns-prefetch" href="//example.com">`

通过简单的一行代码就可以告知那些兼容此特性的浏览器进行DNS预解析，这意味着当浏览器真正请求该域名下的某个资源时，DNS的解析已经完成

**2、预连接preconnect**

与DNS预解析类似，preconnect不仅完成DNS预解析，同时还将进行TCP握手和建立传输层协议

`<link rel="preconnect" href="http://example.com">`

现代浏览器都试着预测网站将来需要哪些连接，然后预先建立socket连接，从而消除昂贵的DNS查找、TCP握手和TLS往返开销。但浏览器还是不够聪明，并不能准确预测每个网站的所有预链接目标。Firefox39和Chrome46已经可以使用preconnect告诉浏览器我们需要进行哪些预连接。

**3、预获取prefetching**

如果我们确定某个资源将来一定会被使用到，可以让浏览器预先请求该资源并放入浏览器缓存中。如一个图片、脚本或任何可以被浏览器缓存的资源

`<link rel="prefetch" href="image.png">`

与DNS预解析不同，预获取真正请求并下载了资源，并储存在缓存中。但预获取还依赖于一些条件，某些预获取可能会被浏览器忽略，例如从一个非常缓慢的网络中获取一个庞大的字体文件

**4、subresources**

预获取资源具有最高的优先级，在所有prefetch项之前进行：

`<link rel="subresource" href="styles.css">`

rel=prefetch 为将来的页面提供了一种低优先级的资源预加载方式

rel=subresource 为当前页面提供了一种高优先级的资源预加载

**5、预渲染prerender**

预先加载文档的所有资源

`<link rel="prerender" href="http://example.com">`

这类似于在一个隐藏的tab页中打开了某个链接：将下载所有资源、创建DOM结构、完成页面布局、应用CSS样式和执行JavaScript脚本等。当用户真正访问该链接时，隐藏的页面就切换为可见，使页面看起来就是瞬间加载完成一样。Google搜索在其即时搜索页面中已经应用该技术多年了，微软也宣称将在IE11中支持该特性。

需要注意的是不要滥用该特性，当你知道用户一定会点击某个链接时才可以进行预渲染，否则浏览器将无条件地下载所有预渲染需要的资源。

### 参考链
[http://bubkoo.com/2015/11/19/prefetching-preloading-prebrowsing/](http://bubkoo.com/2015/11/19/prefetching-preloading-prebrowsing/)
