---
title: 箭头函数和普通函数的区别
categories: base
tags: #文章標籤 可以省略
  - js
  - function
---

#### 1.箭头函数不绑定 this

```js
const Obj = {
  value: 1,
  getValue: function() {
    console.log(this.value)
  },
  // 普通函数 bind this
  doubleVal: function() {
    return function() {
      this.value = this.value * 2;
    }.bind(this);
  },
  // 普通函数 this 赋给一个变量来解决
  doubleVal1: function() {
    const that = this;
    return function() {
      that.value = that.value * 2;
    };
  },
  // 箭头函数自动绑定 this
  doubleVal2: function() {
    return () => {
      this.value = this.value * 2;
    }
  }
}
```

箭头函数体内的 this 对象，就是定义时所在的对象，而不是使用时所在的对象。

#### 2.箭头函数不能使用 new 操作符

```js
const Foo = () => {};
const foo = new Foo();  //Foo is not a constructor
```

箭头函数不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。

#### 3.箭头函数没有原型属性

```js
const Foo = () => {};
console.log(Foo.prototype) //undefined
```

#### 4.箭头函数不绑定 arguments ，取而代之用 rest 参数解决

```js
const foo = (...args) => {
  return args[0]
}
console.log(foo(1))    //1
```

箭头函数不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

#### 5.不能简单返回对象字面量

```js
const func1 = () => {  foo: 1  };
func1();  // undefined

const func2 = () => ({ foo: 1 });
func2();  // { foo: 1 }
```

#### 6.变量提升

```js
obj2(); // obj is not a function
const obj2 = () => { return 'abc'};

obj1() // abc
function obj1() {
  return 'abc';
};
```

#### 7.不可以使用yield命令，因此箭头函数不能用作 Generator 函数
