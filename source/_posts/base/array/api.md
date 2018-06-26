---
title: 数组常用API
categories: base
tags: #文章標籤 可以省略
  - array
---

### filter

filter() 方法创建一个新数组, 其包含通过所提供函数实现的测试的所有元素。

```js
var new_array = arr.filter(callback[, thisArg])

var words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];
const result = words.filter(word => word.length > 6);
// expected output: ["exuberant", "destruction", "present"]
```

### find

find() 方法返回数组中满足提供的测试函数的第一个元素的值。否则返回 undefined。

```js
var array1 = [5, 12, 8, 130, 44];
var found = array1.find(function(element) {
  return element > 10;
});
// expected output: 12
```

### findIndex

findIndex()方法返回数组中满足提供的测试函数的第一个元素的索引。否则返回-1。

```js
var array1 = [5, 12, 8, 130, 44];

function findFirstLargeNumber(element) {
  return element > 13;
}

array1.findIndex(findFirstLargeNumber)
// expected output: 3
```

### forEach

forEach() 方法对数组的每个元素执行一次提供的函数。

```js
array.forEach(callback(currentValue, index, array){
    //do something
}, this)

var array1 = ['a', 'b', 'c'];

array1.forEach(function(element) {
  console.log(element);
});

// expected output: "a"
// expected output: "b"
// expected output: "c"
```

- 没有返回一个新数组，也没有返回值
-  没有办法中止或者跳出 forEach 循环，除了抛出一个异常

### map

map() 方法创建一个新数组，其结果是该数组中的每个元素都调用一个提供的函数后返回的结果。

```js
var array1 = [1, 4, 9, 16];

// pass a function to map
const map1 = array1.map(x => x * 2);

// expected output: Array [2, 8, 18, 32]
```

### every

every() 方法测试数组的所有元素是否都通过了指定函数的测试。

```js
function isBelowThreshold(currentValue) {
  return currentValue < 40;
}

var array1 = [1, 30, 39, 29, 10, 13];

array1.every(isBelowThreshold);
// expected output: true

```

### same

some() 方法测试数组中的某些元素是否通过由提供的函数实现的测试。

```js
var array = [1, 2, 3, 4, 5];

var even = function(element) {
  // checks whether an element is even
  return element % 2 === 0;
};

array.some(even);
// expected output: true
```

### reduce

reduce() 方法对累加器和数组中的每个元素（从左到右）应用一个函数，将其减少为单个值。

arr.reduce(callback[, initialValue])

```js
const array1 = [1, 2, 3, 4];
const reducer = (accumulator, currentValue) => accumulator + currentValue;

// 1 + 2 + 3 + 4
array1.reduce(reducer);
// expected output: 10

// 5 + 1 + 2 + 3 + 4
array1.reduce(reducer, 5);
// expected output: 15
```

### slice

slice() 方法返回一个从开始到结束（不包括结束）选择的数组的一部分`浅拷贝`到一个新数组对象，且原始数组不会被修改。

**语法**

```js
arr.slice();
// [0, end]

arr.slice(begin);
// [begin, end]

arr.slice(begin, end);
// [begin, end)
```

**栗子**

```js
var animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];

animals.slice(2);
// expected output: Array ["camel", "duck", "elephant"]

animals.slice(2, 4);
// expected output: Array ["camel", "duck"]

animals.slice(1, 5);
// expected output: Array ["bison", "camel", "duck", "elephant"]
```

slice 不修改原数组，只会返回一个`浅复制`了原数组中的元素的一个新数组，原数组的元素会按照下述规则拷贝:

- 如果该元素是个对象引用 （不是实际的对象），slice 会拷贝这个对象引用到新的数组里。两个对象引用都引用了同一个对象。如果被引用的对象发生改变，则新的和原来的数组中的这个元素也会发生改变
- 对于字符串、数字及布尔值来说（不是 String、Number 或者 Boolean 对象），slice 会拷贝这些值到新的数组里。在别的数组里修改这些字符串或数字或是布尔值，将不会影响另一个数组
- 如果向两个数组任一中添加了新元素，则另一个不会受到影响

### splice

splice() 方法通过删除现有元素或添加新元素来更改一个数组的内容。

```js
array.splice(start[, deleteCount[, item1[, item2[, ...]]]])
```

- start​: 指定修改的开始位置（从0计数）
- deleteCount: 可选，整数，表示要移除的数组元素的个数。如果 deleteCount 是 0，则不移除元素。deleteCount > 0，表示要删除的元素个数
- item1, item2: 可选，要添加进数组的元素，从start 位置开始。如果不指定，则 splice() 将只删除数组元素。

```js
// 从第2位开始删除1个元素，然后插入“trumpet”
var myFish = ['angel', 'clown', 'drum', 'sturgeon'];
var removed = myFish.splice(2, 1, "trumpet");
//运算后的myFish: ["angel", "clown", "trumpet", "surgeon"]
//被删除元素数组：["drum"]
```

### 参考链接
- (MDN-Array)[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array]
