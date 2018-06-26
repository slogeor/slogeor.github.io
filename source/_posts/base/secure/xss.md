---
title: XSS
categories: base
tags: #文章標籤 可以省略
  - secure
---

### 概述

对于将要放置到 HTML 页面 body 里的不可信数据，进行 HTML 编码已经足够防御 XSS 攻击，甚至将 HTML 编码后的数据放到 HTML 标签的属性（attribute）里也不会产生 XSS 漏洞（但前提是这些属性都正确使用了引号）。

如果将 HTML 编码后的数据放到了 `<Script />` 标签里的任何地方，甚至是 HTML 标签的事件处理属性里、或者是放到了CSS、URL 里，XSS 攻击依然会发生，在这种情况下，HTML 编码不起作用。

### 准则

**1.不要在页面中插入任何不可信数据，除非这些数已经据根据下面几个原则进行了编码**

```js
<script>…不要在这里直接插入不可信数据…</script>
<!– …不要在这里直接插入不可信数据… –>
<div 不要在这里直接插入不可信数据="…"></div>
<div name="…不要在这里直接插入不可信数据…"></div>
<a 不要在这里直接插入不可信数据 href="…"></a>
<style>…不要在这里直接插入不可信数据…</style>
```

**2.将不可信数据插入到 HTML 标签之间时，对这些数据进行 HTML Entity 编码**

```js
<body>…插入不可信数据前，对其进行HTML Entity编码…</body>
<div>…插入不可信数据前，对其进行HTML Entity编码…</div>
<p>…插入不可信数据前，对其进行HTML Entity编码…</p>
以此类推，往其他HTML标签之间插入不可信数据前，对其进行 HTML Entity 编码
```

**3.将不可信数据插入到 HTML 属性里时，对这些数据进行 HTML 属性编码**

```js
<div attr="…插入不可信数据前，进行HTML属性编码…"></div>
```

**4.将不可信数据插入到 SCRIPT 里时，对这些数据进行 SCRIPT 编码**

```js
<script>alert("…插入不可信数据前，进行JavaScript编码…")</script>
<script>x = "…插入不可信数据前，进行JavaScript编码…"</script>
<div onmouseover="x='…插入不可信数据前，进行JavaScript编码…'"</div>
```

特别需要注意在XSS防御中，有些 JavaScript 函数是极度危险的，就算对不可信数据进行 JavaScript 编码，也依然会产生 XSS 漏洞

```js
<script>window.setInterval("…就算对不可信数据进行了JavaScript编码，这里依然会有XSS漏洞…");</script>
```

**5.将不可信数据插入到 style 属性里时，对这些数据进行 CSS 编码**

```js
<style>selector { property : " …插入不可信数据前，进行CSS编码… "} </style>
<span style=" property : …插入不可信数据前，进行CSS编码… ">…</span>
```

**6.将不可信数据插入到 HTML URL 里时，对这些数据进行 URL 编码**

```js
<a href="http://www.abcd.com?param=…插入不可信数据前，进行URL编码…"> Link Content </a>
```

**7.使用富文本时，使用XSS规则引擎进行编码过滤**

### 总结

由于很多地方都可能产生 XSS 漏洞，而且每个地方产生漏洞的原因又各有不同，所以对于 XSS 防御来说，我们需要在正确的地方做正确的事情，即根据不可信数据将要被放置到的地方进行相应的编码。

### 参考链接

- [http://blog.jobbole.com/47372/](http://blog.jobbole.com/47372/)

