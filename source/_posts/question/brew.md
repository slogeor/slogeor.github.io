---
title: brew安装失败
categories: question
tags: #文章標籤 可以省略
  - brewcask
---

#### 问题描述

```
➜  ~ brew cask search qq
==> Exact match
qq
==> Partial matches
qqbrowser qqinput qqmacmgr qqmusic
➜  ~
brew cask install qq
Error: Cask 'qq' definition is invalid: Bad header line: parse failed
```

#### 解决方案

```
brew update; brew cleanup; brew cask cleanup
brew uninstall --force brew-cask; brew update
```
