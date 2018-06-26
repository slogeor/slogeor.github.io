---
title: 性能优化指标
categories: base
tags: #文章標籤 可以省略
  - performance
---

### <font color="red">三个层面</font>

- 浏览器层面：通过预加载，减少DNS解析时间
- 传输层面：减少请求数，降低请求量
- 执行层面：减少重绘、回流，删除冗余代码、资源本地化

### <font color="red">具体方法</font>

- 添加 DNS 预解析 dns-prefetch
- 减少域名个数，控制 5 个以内
- 首屏內联 CSS、JavaScript
- HMTL 代码压缩
- 减少 cookie 大小
- 可以使用 CSS/SVG/ICON Font 替代图片
- 在 HTML 里不要缩放图片
- 移除冗余代码
- 减少页面 layout 次数
- 首屏加载、按需加载、第三方资源异步加载、Lazyload
- 单页应用
- 资源离线、本地数据持久化
- 服务端接口
	- 1.接口合并，减少请求接口数量
	- 2.减少借口数据量
	- 3.缓存更新频率低、重要程度低的接口数据

### <font color="red">想法</font>

- 打点分析数据
- A-B 测试
