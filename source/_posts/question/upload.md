---
title: 文件上传问题
categories: question
tags: #文章標籤 可以省略
  - 文件上传
---

问题： JavaScript 解决文件上传后，不能再上传同名的文件
解决办法：document.getElementById('idSelector').value = "";
