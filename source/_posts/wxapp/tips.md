---
title: 微信小程序开发经验@01
categories: base
tags: #文章標籤 可以省略
  - wxapp
---

### 生成小程序二维码

1.先获取 access_token

接口调用请求，依赖 appid 和 secret。

```js
https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET
```

[点击查看详情说明](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140183)

2.获取二维码接口

```js
// post 请求
https://api.weixin.qq.com/cgi-bin/wxaapp/createwxaqrcode?access_token=ACCESS_TOKEN
```

3.接口参数: 设置扫码进入的路径

```js
{
  "scene":  "&from=code",
  "path": "pages/index?query=1",
  "width": 430,
}
```

4.输出二维码到文件

```js
curl -H "Content-Type:application/json" -X POST -d '{"scene": "&from=code","page": "pages/pichu/pages/tuan/list/index"}' https://api.weixin.qq.com/wxa/getwxacodeunlimit?access_token=10_pONZvNQ0H-ZsowcXftItadVOd3L6grKYrSt1txRFaQhC05EOh0fhLy9tDEcThZnE6TaAQmcKlaMK7PIy4CXx5idL9D7yqGOLHo07siQeLLMRgrNCCQyoR3K9UA9yvJK9WO_OkAMbFqLAuoqmHBIgABAVPW > a.png
```

[官方生成二维码文档链接](https://developers.weixin.qq.com/miniprogram/dev/api/qrcode.html?search-key=%E4%BA%8C%E7%BB%B4%E7%A0%81)

### getUserInfo

第一次用模拟器打开小程序发现登录不上的，会因为获取不了用户信息，在页面添加下面代码，页面渲染完点击一下即可

```
<button open-type="getUserInfo">小姐姐，请点我</button>
```

### 分享

1.模板设置

必须用 `button` 标签，同时要设置 `open-type` 属性，取值是 `share`
```js
<button open-type="share">分享</button>
```

2.逻辑代码

```js
// 显示当前页面的转发按钮
wx.showShareMenu({
  withShareTicket: true
});
```

用户点击分享按钮后触发 Page.onShareAppMessage() 事件

```js
onShareAppMessage() {
  const that = this;
  return {
    title: '分享文案',
    path: '进入的页面路径',
    imageUrl: '自定义图片路径',
    success(e) {
      // 分享成功
      if (e.shareTickets) {
        // 获取分享目标id
        wx.getShareInfo({
          shareTicket: e.shareTickets[0],
          success() {}
        });
      }
    },
    fail() {
      // 分享失败
    }
  };
},
```

[官方分享文档链接](https://developers.weixin.qq.com/miniprogram/dev/api/share.html)


### 分包加载

分包限制:

- 整个小程序所有分包大小不超过 4M
- 单个分包/主包大小不能超过 2M

`app.json`配置如下。

```js
{
  "pages":[
    "pages/index",
    "pages/logs"
  ],
  "subPackages": [
    {
      "root": "pages/agreement/",
      "pages": [
        "agreemenA/index",
        "agreementB/index",
      ]
    },
    {
      "root": "pages/bike/",
      "pages": [
        "bikeA/index",
        "bikeB/index",
      ]
    },
  ]
}
```

[官方分包加载文档](https://developers.weixin.qq.com/miniprogram/dev/framework/subpackages.html?search-key=%E5%88%86%E5%8C%85)
