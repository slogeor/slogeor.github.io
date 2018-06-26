---
title: 数据类型的比较
categories: base
tags: #文章標籤 可以省略
  - js
---

### == VS ===

== 两边值类型不同的时候，要先进行类型转换，再比较。
=== 不做类型转换，类型不同的一定不等。

**==比较**

如果两个类型相同，进行 === 比较；如果类型不等，会先隐式转换然后在进行 == 比较。

基本数据类型之间的比较

null、 undefined、数字、字符串、布尔

```js
null == undefined  // true
1 == '1'          // true
1 == true         // true
true == 1         // true
2 == true         // false
false == 0        // true
0 == false        // true
false == 'false'  // false
true == 'false'   // false
```
布尔类型和数字或字符类型两个等号比较时，true 会先转换成数值 1， false 会先转换成数值 0，然后在进行比较。

**复杂数据类型与基本数据类型之间的比较**

复杂数据类型会先转换成基础类型的值再比较。

复杂数据类型换成基础类型: 利用它的 `toString` 或者 `valueOf` 方法，转换时会 `valueOf` 优先于 `toString`

```js
var obj = {
  toString: function() { return 1},
};

obj == true   // true

var obj1 = {
  toString: function() { return '2'},
  valueOf: function() { return '1'},
};

obj1 == '1'     // true
obj1 == '2'    // false

obj1 == obj   // flase
```

### null  undefined

undefined 和 null 出现在判断条件语句中，会被自动转为 false；相等运算比较也认为两者相等。

null 是一个表示 `无` 的对象，转为数值时为 0

undefined 是一个表示 `无` 的原始值，转为数值时为NaN

null 表示 `没有对象`，即该处不应该有值，典型用法是：

- 1.作为函数的参数，表示该函数的参数不是对象
- 2.作为对象原型链的终点

undefined 表示 `缺少值`，就是此处应该有一个值，但是还没有定义，典型用法是：

- 1.变量被声明了，但没有赋值时，就等于undefined
- 2.调用函数时，应该提供的参数没有提供，该参数等于undefined
- 3.对象没有赋值的属性，该属性的值为 undefined
- 4.函数没有返回值时，默认返回 undefined

### toString VS valueOf

如果对象的 toString 和 valueOf 都存在的时候，对象转换为字符串的时候会优先考虑 `valueOf` 方法，对象输出操作会优先考虑 `toString` 方法，整体来看 `valueOf` 优于 `toString`。

```js
var obj = {
  toString: function() { return '12'; },
  valueOf: function() { return '10'; }
}

alert(obj);           //12  toString
alert('' + obj);      //10  valueOf
alert(+obj);          //10  valueOf
alert(String(obj));   //12 toString
alert(Number(obj));   //10 valueOf
alert(obj == '10');   //true valueOf
alert(obj === '12');  //false
```

### 测试题

定义变量 a，执行下面代码能够输出 `我走进来了`。

```js
if(a == 1 && a == 2 && a == 3){
  console.log("我走进来了");
}
```
