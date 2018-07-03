---
title: 支付宝小程序开发经验@01
categories: base
tags: #文章標籤 可以省略
  - aliapp
---

### 支付宝小程序抓包

使用支付宝扫描链接二维码，二维码内容为（`https://ds.alipay.com/fd-ipddfamm/index.html`），点击`关闭安全代理`按钮，关闭安全代理，通过此方式可在手机上进行抓包。

![img](https://github.com/slogeor/images/blob/master/fe/2018/06/04.png?raw=true)

### 配置扫码拦截

![img](https://github.com/slogeor/images/blob/master/fe/2018/06/05.png?raw=true)

- 1.二维码地址: 配置的url `http://slogeor.com/f`不能存在重定向，如果存在，配置的时候可以先把 ng 重定向关闭，配置好后再打开。
- 2.下载校验文件
- 3.存放校验文件: 将校验文件放到`http://slogeor.com/`对应服务器的根目录
- 4.点击这里: 如果验证通过，就可以配置拦截 url 进入的页面，否则不能配置页面
