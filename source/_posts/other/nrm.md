---
title: nrm -- NPM registry 管理工具
categories: other
tags: #文章標籤 可以省略
  - nrm
---

通过 nrm 可以方便的查看和切换当前使用的registry。

### 安装

```js
$ npm install -g nrm
```

### 查看 registry

```js
~ nrm ls

npm ---- https://registry.npmjs.org/
cnpm --- http://r.cnpmjs.org/
* taobao - https://registry.npm.taobao.org/
nj ----- https://registry.nodejitsu.com/
rednpm - http://registry.mirror.cqupt.edu.cn/
npmMirror  https://skimdb.npmjs.com/registry/
edunpm - http://registry.enpmjs.org/
snpm --- http://slogeor.com/
```

### 切换

```js
~ nrm use taobao
  Registry has been set to: https://registry.npm.taobao.org/
```

### 新增

```js
nrm add snpm http://slogeor.com/
```

### 其他命令

```js
nrm help // show help
nrm list // show all registries
nrm use cnpm // switch to cnpm
nrm home // go to a registry home page
```
