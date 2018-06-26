---
title: HTTP Headers
categories: base
tags: #文章標籤 可以省略
  - http
---

### Request Header

- Host
- User-Agent
- Accept
- Accept-Encoding
- Accept-Language
- Accept-Charset
- Cache-Control
- Cookie
- Connection: Close

```js
Host: www.baidu.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_1) ...
Accept: text/html,application/xhtml+xml;image/webp,image/apng;q=0.8...
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,ja;q=0.7
Cache-Control: max-age=0
Cookie: BAIDUID=E364865F44E573FD6858F41329938092:FG=1;
Connection: keep-alive
```

### Response Header

- Server
- Connection: keep-alive/Close
- Content-type
- Content-Length
- Date
- Expires
- Last-Modified
- Etag
- Cache-Control
- Transfer-Encoding

```js
Server: Apache
Cache-Control: max-age=315360000
Content-Length: 40872
Content-Type: image/png
Date: Sun, 17 Jun 2018 01:48:44 GMT
Etag: "9fa8-56ebc9ab61427"
Expires: Wed, 14 Jun 2028 01:48:44 GMT
Last-Modified: Sat, 16 Jun 2018 06:40:12 GMT
```
