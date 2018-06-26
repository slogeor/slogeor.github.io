---
title: Redux 高级
categories: note
tags: #文章標籤 可以省略
     - redux
---

### 异步 action

在基础教程里，我们的 action 都是同步的，每当 dispatch action 时，state 会被立即更新。

当调用异步 API 时，有两个非常关键的时刻：发起请求的时刻，和接受到响应的时刻。这两个时刻都可能更改应用的 state。

一般情况下，每个 API 请求都需要 dispatch 至少三种 action。

* 一种通知 reducer 请求开始的 action
* 一种通知 reducer 请求成功的 action
* 一种请求 reducer 请求失败的 action

##### 异步 action 创建函数

将同步的 action 创建函数和网络请求结合起来，标准的做法是使用 Redux Thunk 中间件。通过使用指定的 middleware，action 创建函数除了返回 action 对象外，还可以返回函数，这个 action 创建函数就成为了 thunk。

当 action 创建函数返回函数时，这个函数会被 Redux Thunk middleware 执行。这个函数并不需要保持纯净；这个函数还可以 dispatch action，就像 dispatch 同步的 action 一样。

##### thunk action 创建函数

虽然内部操作不同，可以像其它 action 创建函数一样使用 `store.dispatch(fetchPosts('reactjs'))`

```js
export function fetchPosts(subreddit) {
	// Thunk middleware 知道如何处理函数
	// 这里把 dispatch 方法通过参数的形式传给函数
	// 以此来让它自己也能 dispatch action

	return function (dispatch) {
		// 首次 dispatch，更新应用的 state
		dispatch(requestPosts(subreddit))

		// thunk middleware 调用的函数有返回值，
		// 它会被当作 dispatch 方法的返回值传递

	return fecth(`/url/${subreddit}.json`)
		.then(response => response.json)
		.then(json =>
			// 可以多次 dispatch
			dispatch(receivePosts(subreddit, json))
		)
	}
}
```

thunk 的一个优点是它的结果可以被多次 dispatch。

### Middleware

默认情况下，createStore() 所创建的 Redux store 没有使用 middleware，所以只支持同步数据流。但可以使用 applyMiddleware() 来增强 createStore()。

像 redux-thunk、redux-promise 这样支持异步的 middleware 都包装了 store 的 dispatch() 方法，以此来让你 dispatch 一些除了 action 以外的其他内容，例如函数或者 promise。可以使用任何 middleware 以自己的方式解析你的 dispatch 的任何内容，并继续传递 actions 给下一个 middleware。

当 middleware 链中的最后一个 middleware 开始 dispatch action 时，这个 action 必须是普通对象。这是同步式 Redux 数据流开始的地方。

相对于 Express 或者 Koa 的 middleware，Redux middleware 被用于解决不同的问题，它提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。可以利用 Redux middleware 来进行日志记录、创建奔溃报告、调用异步接口或者路由等。 eg: 日志记录。

##### 封装 Dispatch

```js
function dispatchAndLog(store, action) {
	console.log('dispatching', action);
	stroe.dispatch(action);
	console.log('next state', store.getState())
}

dispatchAndLog(store, addTodo('Use Redux'));
```

##### Monkeypatching

可以直接替换 store 实例中的 dispatch 函数。

```js
let next = store.dispatch;

store.dispacth = function dispatchAndLog(action) {
	console.log('dispatching', action);
	let result = next(action);
	console.log('next state', store.getState());
	return result
}
```

##### 隐藏 Monkeypatching

Monkeypatching 本质上是一种 hack，将任意的方法替换成你想要的。这种替换的本质是用我们自己的函数替换掉 store.dispatch。如果不这样做，而是在函数中返回dispatch呢

```js
function logger(store) {
	let next = store.dispatch();

	// store.dispatch = function dispatchAndLog(action) {

	return function dispatchAndLog(action) {
		console.log('dispatching', action)
		let result = next(action)
		console.log('next state', store.getState())
		return result;
	}
}
```
我们可以在 Redux 内部提供一个可以将实际的 monkeypatching 应用到 store.dispatch 中辅助方法：

```js
function applyMiddlewareByMonkeypatching(store, middlewares) {
	middlewares = middlewares.slice();
	middlewares.reverse();

	// 每一个 middleware 中变换 dispatch 方法
	middlewares.forEach(middleware =>
		store.dispatch = middleware(store)
	)
}

applyMiddlewareByMonkeypatching(store, [ logger, crashReporter ])
```

##### 移除 Monkeypatching

可以实现每一个 middleware 都可以操作前一个 middleware 包装过的 store.dispatch

```js
function logger(store) {
	// 这里的 next 必须指向前一个 middleware 返回的函数
	let next = store.dispatch;

	return function dispatchAndLog(action) {
		console.log('dispatching', action)
		let result = next(action)
		console.log('next state', store.getState())
		return result;
	}
}
```

这样将 middleware 串连起来的必要性是显而易见的。 如果 applyMiddlewareByMonkeypatching 方法中没有在第一个 middleware 执行时立即替换 store.dispatch，那么 store.dispatch 将会一直指向原始的 dispatch 方法。

可以让 middleware 以方法参数的形式接收一个 next() 方法，而不是通过 store 的实例去获取。

```js
function logger(store) {
	return function wrapDispatchAndLog(next) {
		console.log('dispatching', action)
		let result = next(action)
		console.log('next state', store.getState())
		return result;
	}
}
```

等价于

```js
const logger = store => next => action => {
	console.log('dispatching', action)
	let result = next(action)
	console.log('next state', store.getState())
	return result
}
```

这正是 Redux middleware 的样子。接收一个 next() 的 dispatch 函数，并返回一个 dispatch 函数，返回的函数会被作为下一个 middleware 的 next()。

##### 单纯的使用 Middleware

```js
// 这只是一种单纯的实现方式
function applyMiddleware(store, middlewares) {
	middlewares = middlewares.slice();
	middlewares.reverse();

	let dispatch = stroe.dispatch;

	middlewares.forEach(middleware =>
		dispatch = middleware(store)(dispatch)
	)

	return Object.assign({}, store, { dispatch })

}
```

* 只暴露了 store API 的子集给 middleware：dispatch(action) 和 getState()
* 确保如果你在 middleware 中调用的是 store.dipatch(action)，而不是 next(action)
* 为了保证只能应用 middleware 一次，它作用在 createStore() 上而不是 store 本身

##### 终极方法

```js
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

import {createStore, combineReducers, applyMiddleware} from 'redux';

let todoApp = combineReduce(reducers);

let store = createStore(
	todoApp,
	// appleMiddleware() 告诉 createStore() 如何处理中间件
	applyMiddleware(logger, crashReporter);
)

// 将经过 logger、crashReporter 两个 middleware
store.dispatch(addTodo('Use Redux'))
```
