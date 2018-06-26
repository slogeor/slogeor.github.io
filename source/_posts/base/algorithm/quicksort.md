---
title: 快速排序
categories: base
tags: #文章標籤 可以省略
  - algorithm
---

快速排序使用分治法（Divide and conquer）策略来把一个序列（list）分为两个子序列（sub-lists）

**步骤为：**

- 1.基准: 从数列中挑出一个元素，称为"基准"（pivot）
- 2.分区: 重新排序数列，所有元素比基准值小的放在基准前面，所有元素比基准值大的放在基准的后面（相同的数可以到任一边），在这个分区结束之后，该基准就处于数列的中间位置。这个称为分区（partition）操作
- 3.递归: 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序
- 4.结束条件: 递归的最底部情形，是数列的大小是零或一，也就是永远都已经被排序好了。

虽然一直递归下去，但是这个算法总会结束，因为在每次的迭代（iteration）中，它至少会把一个元素放到它最后的位置去。

在简单的伪代码中，此算法可以被表示为：

```js
function quicksort(q) {
  let list less, pivotList, greater
  if length(q) <= 1 {
    return q
  } else {
    select a pivot value pivot from q
    for each x in q except the pivot element
    if x < pivot then add x to less
    if >= pivot then add x to greater
    add pivot to pivotList
    return concatenate(quicksort(less), pivotList, quicksort(greater))
  }
}
```

**数组第一次排序操作图**

![img](https://github.com/slogeor/images/blob/master/fe/2018/06/03.png?raw=true)

```js
Array.prototype.quickSort = function() {
  const len = this.length;

  if (len <= 1) {
    return this.slice(0);
  }

  const left = [];
  const right = [];
  const mid = [this[0]];

  for (let i = 1; i < len; i++) {
    const val = this[i];
    this[i] < mid[0] ? left.push(val) : right.push(val);
  }

  return left.quickSort().concat(mid.concat(right.quickSort()));
};

var arr = [5, 3, 7, 4, 1, 9, 8, 6, 2];
arr = arr.quickSort();
```


