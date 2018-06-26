---
title: Promise
categories: base
tags: #文章標籤 可以省略
  - promise
  - js
---

**Promise对象有以下两个特点**

1.对象的状态不受外界影响
  - pending: 进行中
  - fulfilled: 已成功
  - rejected: 已失败

2.一旦状态改变，就不会在变化，任何时候都可以得到这个结果
  - pending --> fulfilled
  - pending --> rejected

如果改变已经发生，再对 Promise 对象添加回调函数，会立即得到这个结果，这与事件完全不同，事件的特点是，如果错过了，再去监听也得不到结果。

Promise 的缺点

- 一旦新建 Promise，就无法取消
- 如果不指定 Promise 的回调函数，内部的错误无法抛到外部
- 当 Promise 处于 pending 状态时，无法得知目前进行到哪个阶段

#### Promise 基本用法

ES6 规定，Promise对象是一个构造函数，用来生成Promise实例。

```js
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

Promise 新建后，就会立即执行

```js
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// resolved
```

一般来说，调用 resolve 或 reject 以后，Promise 的使命就完成了，后继操作应该放到 then 方法里面，而不应该直接写在 resolve 或 reject 的后面。所以，最好在它们前面加上 return 语句，这样就不会有意外。

```js
new Promise((resolve, reject) => {
  resolve(1);
  console.log(2);
}).then(r => {
  console.log(r);
});
// 2
// 1

new Promise((resolve, reject) => {
  return resolve(1);
  console.log(2);
}).then(r => {
  console.log(r);
});
// 1
```

简单的栗子

```js
const p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

const p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail
```

p1 是一个 Promise，3 秒之后变为 rejected。p2 的状态在 1 秒之后改变，resolve 方法返回的是 p1。由于 p2 返回的是另一个 Promise，导致p2 自己的状态无效了，由 p1 的状态决定 p2 的状态。

所以，后面的 then 语句都变成针对后者（p1）。又过了 2 秒，p1 变为 rejected，导致触发 catch 方法指定的回调函数。

#### Promise.prototype.then()

Promise 实例具有 then 方法，也就是说，then 方法是定义在原型对象 Promise.prototype 上的。

它的作用是为 Promise 实例添加状态改变时的回调函数。

then 方法返回的是一个新的 Promise 实例，可以采用链式写法，即 then 方法后面再调用另一个 then 方法。

采用链式的 then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数返回一个 Promise 对象，这时后一个回调函数，就会等待该 Promise 对象的状态发生变化，才会被调用。

```js
const p1 = new Promise(function(resolve) {
  setTimeout(() => {
    return resolve('step1');
  }, 1000)
});


const p2 = function(val) {
  return new Promise(function(resolve) {
    setTimeout(() => {
      return resolve('step2' + '-' + val);
    }, 1000)
  })
};

p1.then((v1) => {
  return p2(v1)
}).then((v2) => {
  console.log(v2)
})

// step2-step1
```

#### Promise.prototype.catch()

Promise.prototype.catch 方法是 `.then(null, rejection)` 的别名，用于指定发生错误时的回调函数。

```js
const p1 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    resolve('111');
  }, 1000);
});

p1.then((val) => {
  console.log('resolve:',val);
  console.log(a)
}).catch((err) => {
  console.log('reject:', err)
})

// resolve: 111
// reject: ReferenceError: a is not defined
```

then 方法指定的回调函数，如果运行中抛出错误，也会被 catch 方法捕获。

Promise 对象的错误具有`冒泡`性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个 catch 语句捕获。

```js
const p1 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    reject(new Error('error'));
  }, 1000);
});

const p2 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    resolve('111');
  }, 1000);
});

p1.then((val1) => {
  console.log('step1 then');
  console.log('resolve:',val1);
  return p2;
})
.then((val2) => {
  console.log('step2 then');
  console.log('resolve:',val2);
})
.catch((err) => {
  console.log('reject:', err)
});

// reject: Error: error
```

如果没有使用 catch 方法指定错误处理的回调函数，Promise 对象抛出的错误不会传递到外层代码，就是说 Promise 内部的错误不会影响到 Promise 外部的代码，通俗的说法就是 `Promise 会吃掉错误`。

一般总是建议，Promise 对象后面要跟 catch 方法，这样可以处理 Promise 内部发生的错误。catch 方法返回的还是一个 Promise 对象，因此后面还可以接着调用 then 方法。

```js
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行会报错，因为x没有声明
    resolve(x + 2);
  });
};

someAsyncThing()
.catch(function(error) {
  console.log('oh no', error);
})
.then(function() {
  console.log('carry on');
});
```

#### Promise.prototype.finally()

finally 方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。该方法是 ES2018 引入标准的。

finally 方法的回调函数不接受任何参数，这意味着没有办法知道，前面的 Promise 状态到底是 fulfilled 还是 rejected。这表明，finally 方法里面的操作，应该是与状态无关的，不依赖于 Promise 的执行结果。

finally 本质上是 then 方法的特例。

```js
promise
.finally(() => {
  // 语句
});

// 等同于
promise
.then(
  result => {
    // 语句
    return result;
  },
  error => {
    // 语句
    throw error;
  }
);
```
实现也很简单。

```js
Promise.prototype.finally = function (callback) {
  const P = this.constructor;
  return this.then(
    value  => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => { throw reason })
  );
};
```

#### Promise.all()

Promise.all 方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。

```js
const p = Promise.all([p1, p2, p3]);
```

p 的状态由p1、p2、p3 决定，分成两种情况。

- 只有 p1、p2、p3 的状态都变成 fulfilled，p 的状态才会变成 fulfilled，此时 p1、p2、p3 的返回值组成一个数组，传递给 p 的回调函数
- 只要 p1、p2、p3 之中有一个被 rejected，p 的状态就变成 rejected，此时第一个被 reject 的实例的返回值，会传递给 p 的回调函数。

```js
const p1 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    resolve('p1');
  }, 1000);
});

const p2 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    resolve('p2');
  }, 1000);
});

const p3 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    resolve('p3');
  }, 1000);
});

const p = Promise.all([p1, p2, p3]);
p.then((val) => {
  console.log('then:',val);
}).catch((err) => {
  console.log('catch:',err);
})
// then: (3) ["p1", "p2", "p3"]



const p1 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    resolve('p1');
  }, 1000);
});

const p2 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    reject('p2');
  }, 1000);
});

const p3 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    resolve('p3');
  }, 1000);
});

const p = Promise.all([p1, p2, p3]);
p.then((val) => {
  console.log('then:',val);
}).catch((err) => {
  console.log('catch:',err);
})

// catch: p2
```

如果作为参数的 Promise 实例，自己定义了 catch 方法，那么它一旦被 rejected，并不会触发 `Promise.all()` 的 catch 方法。

```js
const p1 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    resolve('p1');
  }, 1000);
});

const p2 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    reject('p2');
  }, 1000);
}).catch((err) => {
    console.log('p2 catch:', err)
});

const p3 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    resolve('p3');
  }, 1000);
});

const p = Promise.all([p1, p2, p3]);
p.then((val) => {
  console.log('then:',val);
}).catch((err) => {
  console.log('catch:',err);
})

// p2 catch: p2
// then: (3) ["p1", undefined, "p3"]
```

p2 首先会 rejected，但是 p2 有自己的 catch 方法，该方法返回的是一个新的 Promise 实例，p2 指向的实际上是这个实例。该实例执行完 catch方法后，也会变成 resolved。

#### Promise.race()

Promise.race方法同样是将多个 `Promise` 实例，包装成一个新的 `Promise` 实例。

```js
const p = Promise.race([p1, p2, p3]);
```

上面代码中，只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。

#### Promise.resolve()

有时需要将现有对象转为 Promise 对象，Promise.resolve方法就起到这个作用。

```js
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

#### Promise.reject()
