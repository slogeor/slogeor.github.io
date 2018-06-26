---
title: 如何使用NPM
# categories: other
tags: #文章標籤 可以省略
  - npm
---

#### 1.通过设置全局的npm配置

```js
npm config set registry https://registry.xxx.com/
```

- 优点: 简单方便
- 缺点: 同时使用其他镜像源会比较麻烦

#### 2.npm命令中设置--registry参数

```js
npm install --registry=https://registry.xxx.com/
```

此方法麻烦但准确。


#### 3.配置.npmrc文件

在项目根目录下放一个 `.npmrc` 配置文件，在里面加一行配置registry地址:

```js
registry=https://registry.xxx.com/
```

需要提交这个文件，方便团队其他小伙伴保持统一，推荐这样做。

#### 4.设置别名

```js
alias xnpm="npm --userconfig=$HOME/.xnpmrc"
```
然后在这个 .xnpmrc 文件里增加方法三里的 registry 地址配置，推荐这样做。

#### 5.使用其他第三方的npm源配置工具

这里不再交待
