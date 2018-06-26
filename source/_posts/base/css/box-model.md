---
title: CSS盒模型
categories: base
tags: #文章標籤 可以省略
  - css
---

<b>描述</b>

文档中的每个元素被描绘为矩形盒子，确定其大小，属性，比如颜色、背景、边框，及其位置是渲染引擎的目标

盒模型就是确定元素内容所占用的空间，盒子有四个边界：外边距边界 margin edge、边框边界 border edge、内边距边界 padding edge 与 内容边界 content edge

<b>W3C盒模型</b>

在W3C盒模型中，一个块级元素的总宽度按照如下的方程式计算：

总宽度 = margin-left + border-left + padding-left + width + padding-right + border-right + margin-right

对于高度也使用同样的计算方法，但为了简便起见从现在开始只说 width

<b>IE盒模型</b>

IE盒模型的计算方式和W3C的很相似，但有一点是非常不同的，就是填充和边框并不被包含在计算的范围内

总宽度 = margin-left + width + margin-right

<b>盒模型图</b>

![box_models](https://github.com/slogeor/images/blob/master/fe/2018/04/box_models.gif?raw=true
)

<b>box-sizing</b>

box-sizing 是 CSS3 的一个属性，与盒模型有着千丝万缕的联系

**属性值: content-box**

padding 和 border 不被包含在定义的 width 和 height 之内。对象的实际宽度等于设置的 width 值和border、padding 之和，即Element width = width + border + padding 此属性表现为标准模式下的盒模型

![content-box-2](https://github.com/slogeor/images/blob/master/fe/2018/04/content-box-2.png?raw=true)

**属性值: border-box**

padding 和 border 被包含在定义的 width 和 height 之内。对象的实际宽度就等于设置的 width 值，即使定义有 border 和 padding 也不会改变对象的实际宽度，即 Element width = width 此属性表现为怪异模式下的盒模型

![border-box-2](https://github.com/slogeor/images/blob/master/fe/2018/04/border-box-2.png?raw=true)
