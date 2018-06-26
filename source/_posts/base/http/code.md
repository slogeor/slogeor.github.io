---
title: HTTP Code
categories: base
tags: #文章標籤 可以省略
  - http
---

### 1XX: 信息

- 100 Continue: 服务器收到部分请求，只要服务器没有拒绝该请求，客户端会继续发送其余请求
- 101 Switching Protocols: 服务器转化协议，遵从客户的请求转换到另外一种协议

### 2XX: 成功

- <font color="blue">200 OK: 请求成功</font>
- 201 Created: 请求被创建完成，同时新的资源被创建
- 202 Accepted: 供处理的请求已被接受，但是处理未完成
- 203 Non-authoritative Information 文档已经正常地返回，但一些应答头可能不正确，因为使用的是文档的拷贝
- 204 No Content: 没有新文档。浏览器应该继续显示原来的文档。如果用户定期地刷新页面，而Servlet可以确定用户文档足够新，这个状态代码是很有用的
- 205 Reset: Content 没有新文档。但浏览器应该重置它所显示的内容。用来强制浏览器清除表单输入内容
- 206 Partial Content: 客户发送了一个带有Range头的GET请求，服务器完成了它

### 3XX: 重定向

- 300 Multiple Choices: 多重选择。链接列表。用户可以选择某链接到达目的地。最多允许五个地址
- <font color="blue">301 Moved Permanently: 所请求的页面已经转移至新的url</font>
- <font color="blue">302 Found: 所请求的页面已经临时转移至新的url</font>
- 303 See Other: 所请求的页面可在别的url下被找到
- <font color="blue">304 Not Modified: 未按预期修改文档。客户端有缓冲的文档并发出了一个条件性的请求</font>
- 305 Use Proxy: 客户请求的文档应该通过Location头所指明的代理服务器提取
- 306 Unused: 此代码被用于前一版本。目前已不再使用，但是代码依然被保留
- 307 Temporary Redirect: 被请求的页面已经临时移至新的url

### 4XX: 客户端错误

- 400 Bad Request: 服务器未能理解请求
- 401 Unauthorized: 被请求的页面需要用户名和密码
- 402 Payment Required: 此代码尚无法使用
- <font color="blue">403 Forbidden: 对被请求页面的访问被禁止</font>
- <font color="blue">404 Not Found: 服务器无法找到被请求的页面</font>
- 405 Method Not Allowed: 请求中指定的方法不被允许
- 406 Not Acceptable: 务器生成的响应无法被客户端所接受
- 407 Proxy Authentication Required: 用户必须首先使用代理服务器进行验证，这样请求才会被处理
- 408 Request Timeout: 请求超出了服务器的等待时间
- 409 Conflict: 由于冲突，请求无法被完成
- 410 Gone: 被请求的页面不可用
- 411 Length Required "Content-Length": 未被定义。如果无此内容，服务器不会接受请求
- 412 Precondition Failed: 请求中的前提条件被服务器评估为失败
- 413 Request Entity Too Large: 由于所请求的实体的太大，服务器不会接受请求
- 414 Request-url Too Long: 由于url太长，服务器不会接受请求。当post请求被转换为带有很长的查询信息的get请求时，就会发生这种情况
- 415 Unsupported Media Type: 由于媒介类型不被支持，服务器不会接受请求
- 416   服务器不能满足客户在请求中指定的Range头
- 417 Expectation Failed

### 5XX: 服务器错误

- <font color="blue">500 Internal Server Error: 请求未完成。服务器遇到不可预知的情况</font>
- 501 Not Implemented: 请求未完成。服务器不支持所请求的功能
- 502 Bad Gateway: 请求未完成。服务器从上游服务器收到一个无效的响应
- <font color="blue">503 Service Unavailable: 请求未完成。服务器临时过载或当机</font>
- <font color="blue"> 504 Gateway Timeout: 网关超时</font>
- <font color="blue"> 505 HTTP Version Not Supported: 服务器不支持请求中指明的HTTP协议版本</font>

![img](https://github.com/slogeor/images/blob/master/fe/2016/js/http.code.png?raw=true)

### 参考链接

- [http://www.w3school.com.cn/tags/html_ref_httpmessages.asp](http://www.w3school.com.cn/tags/html_ref_httpmessages.asp)
- [http://www.infoq.com/cn/news/2015/12/how-to-choose-http-status-code](http://www.infoq.com/cn/news/2015/12/how-to-choose-http-status-code)
