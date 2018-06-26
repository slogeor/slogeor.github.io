---
title: webpack找不到entry
categories: question
tags: #文章標籤 可以省略
    - webpack
---

#### 问题提示

Module not found: Error: a dependency to an entry point is not allowed

#### 问题出现

webpack.config.js 配置如下

```
module.exports = {
    //...
    entry: {
        "./home/index.js": "./home/index.js",
        "./poi/index.js": "./poi/index.js"
    },
    //...
};
```

./home/index.js 有如下代码

```
require("../poi/index.js");
```

执行 webpack 命令会报如下错

```
ERROR in ./src/campass/scripts/pages/home/index.js
Module not found: Error: a dependency to an entry point is not allowed
```

#### 原因分析

``` ./home/index.js ``` 不能 require 出现在 ``` module.exports ``` 里 entry 配置的文件
