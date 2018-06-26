### Flux 流程

![img1](https://pic3.zhimg.com/v2-f719fccae7b22258f97c2c3f9490f3f2_b.png)

Flux的整个流程分为四个部分。

* View： 视图层
* Action（动作）：视图层发出的消息（比如mouseClick）
* Dispatcher（派发器）：用来接收Actions、执行回调函数
* Store（数据层）：用来存放应用的状态，一旦发生变动，就提醒Views要更新页面

Redux 作为 Flux 的实现，加入了一些函数式的编程思想，其中最主要的有三点。

* 数据的单一来源：即整个APP尽量不使用内部state，所有的状态全部源自于Redux
* 纯函数reducer：处理action并生成新的state，其形式为 newState = f(state，action)，不允许任何副作用
* 不可变state：只有当state对象被整体替换时候，才会触发view层的更新

### Reducer

```js
function reducer(state = initialState, action) {
  switch (action.type) {
    case SOME_ACTION:
      return Object.assign({}, state, {
        someKey: action.payload
      })
    default:
      return state
  }
}
```

![img0](https://pic2.zhimg.com/v2-404460ceece985d433e1ed1f36cd4215_b.jpg)

### Redux/Redux-thunk

![img2](https://pic2.zhimg.com/v2-8d35f1b5141153260700d1bcdc34d839_b.png)

这种模式除了开发模式简单，几乎没有任何优点，简单的代价就是开发者需要自己动手处理各种dirty things。

* 所有的异步请求全部放在ActionCreator中，大量的异步回调地狱让开发者和维护者都心力憔悴
* 没有使用任何其他的副作用处理框架，实现诸如Async call/cancel非常的困难，需要开发人员大量手工编码，经常会出现先发的请求后处理，导致显示错误的旧数据

### Redux/Redux-Saga

![img03](https://pic1.zhimg.com/v2-1a61fe2319a615056e3573e51b3367e4_b.png)

Redux-Saga充分利用了ES6的Generator特性，一切异步调用都可以通过yield交给Redux-Saga去处理，并在异步代码执行完成后返回yield处继续执行，从此和回调形式的异步调用说再见，同时支持try/catch语法，可以更方便的对代码块进行异常捕获。

### No-Reducer开发模式

![img04](https://pic2.zhimg.com/v2-e8b6aba6784e7025b10e74e02f180a7d_b.png)

简单Action类型定义过多的话，上百的Action类型也会让后来者看得眼花缭乱，不利于代码的维护。为了解决这个问题，我们使用了No-Reducer开发模式

* 简化reducer，仅接收和处理SET_STATE、REPLACE_STATE两种Action，且最终的状态变换一定要调用这两个方法之一
* 简单的Action不设立单独的Action类型，如弹出窗口开关，简单的加减等Action，直接调用SET_STATE完成状态变换
* 复杂的Action都需要有单独的类型定义，如需要从后台获取数据，或者处理步骤超过10行的Action。并且处理函数要求在Saga中独立注册，以供复用或组合

```js
function noReducer(state = initialState, action) {
  switch (action.type) {
    case SET_STATE:
      return Object.assign({}, state, action.state)
    case REPLACE_STATE:
      return action.state
    default:
      return state
  }
}
```

No-Reducer开发模式就是一个增强版的内部state，setState模式。前文说过，Redux推崇数据单一来源，Redux的state原本就是被设计取代组件内部state的，所有组件渲染需要的state都应该被放入Redux中集中管理。而SET_STATE的表现和setState几乎一致，可以让开发者更容易接收Redux，并且付出很小的代价将内部state迁移到Redux中统一管理。



### 参考链接

* [Redux的副作用处理与No-Reducer开发模式](https://zhuanlan.zhihu.com/p/28796342)
* [React、Redux与复杂业务组件的复用](https://juejin.im/post/59b4eb216fb9a00a616f07a3)






