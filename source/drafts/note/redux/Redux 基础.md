---
title: Redux 基础
categories: note
tags: #文章標籤 可以省略
     - redux
---

### 要点

*  应用中所有的 state 都以一个对象树的形式存在一个单一的 store中
*  唯一改变 state 的办法就是触发 action，一个描述发生什么的对象
*  为了描述 action 如何改变 state 树，你需要编写 reducers

把要做的修改变成一个普通对下，这个对象被叫做 action，只有通过 action 才能修改 state。` {type: 'TODO', text: 'abc'}) `

编写专门的函数来决定每个 action 如何改变应该的 state，这个函数叫做 reducer。`(state, action) => state`

#### 三大原则

* 单一数据源
* store 是只读的
* 使用纯函数来执行修改

##### 单一数据

整个应用的 state 被存储在一棵 object tree 中，并且这个 object tree 只存在唯一一个 store 中

```js
console.log(store.getStore())
// output
{
	visibilityFilter: 'SHOW_ALL',
	todos: [
		{
			text: 'Consider using redux',
			completed: true,
		}
	]
}
```

##### store 是只读的

唯一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象

```js
store.dispatch({
	type: 'COMPLITE_TODO',
	index: 1,
});
```

##### 使用纯函数来执行修改

为了描述 action 如何改变 state tree，你需要编写 reducers。

```js
function visibilityFilter(state = 'SHOW_ALL', action) {
	switch(action.type) {
		case: 'SET_VISIBILITY_FILTER':
			return action.filter
		default:
			return state
	}
}

function todos(state = [], action) {
	//...
}

import {combineReducers, createStore } from 'redux';
let	 reducer = combineReducers({
	visibilityFilter,
	todos,
});
let store = createStore(reducer);
```
### action

action 是把数据从应用传到 store 的有效载荷，它是 store 数据的唯一来源。一般会通过 `store.dispatch()` 将 action 传到 store。

添加新 todo 任务的 action 是这样。

```js
const ADD_TODO = 'ADD_TODO';

{
	type: ADD_TODO,
	text: 'Bulid my first app'
}
```

action 内必须使用一个字符串类型的 type 字段来表示将要执行的动作。

##### action 创建函数

action 创建函数就是生成 action 的方法，action 和 action 创建函数这两个概念容易混淆，使用时最好注意区分。

在 Redux 中的 action 创建函数只是简单的返回一个 action。

```js
function addTodo(text) {
	return {
		type: ADD_TODO,
		text,
	}
}
```

Redux 只需要把 action 创建函数的结果传给 dispatch() 方法即可发起一次 dispatch 过程。

```js
dispatch(addTodo(text))
```

或者创建一个被绑定的 action 创建函数来自动 dispatch

```js
const boundAddTodo = (text) => dispatch(addTodo(text))

boundAddTodo(text);
```

store 里能直接通过 store.dispatch() 调用 dispatch() 方法，但是多数情况会使用 react-redux 提供的 connect() 帮转器来调用。bindActionCreators() 可以自动把多个 action 创建函数绑定到 dispatch() 方法上。

### reducer

action 只是描述了有事情发生这一事实，并没有指明应用如何更新 state。而正是 reducer 要做的事。

##### action 处理

reducer 是一个纯函数，接收旧的 state 和 action，返回新的 state。

```js
(previousState, action) => newState
```

保持 reducer 纯净非常重要，永远不要在 reducer 里做这些操作：

* 修改传入参数
* 执行有副作用的操作，如 API 请求和路由跳转
* 调用非纯函数

```js
function todoApp (state = initialState, action) {
	switch(action.type) {
		case SET_VISIBILITY_FILTER:
		return Object.assign({}, state, {
			visibilityFilter: action.filter
		})

		default:
		 	return initialState
	}
}
```

##### 注意：

* 不要修改 state
* 在 default 情况返回旧的 state

##### 拆分 reducer

我们可以开发一个函数来做为主 reducer，它调用多个子 reducer 分别处理 state 中的一部分数据，然后再把这些数据合成一个大的单一对象。主 reducer 并不需要设置初始值时完整的 state。初始时，如果传入 undefined，子 reducer 将负责返回它们的默认值。

```js
function todos(state = [], action) {
	switch(action.type) {}
}

function visibilityFilter(state = SHOW_ALL, action) {
	switch(action.type) {}
}

function todoApp(state = {}, action) {
	return {
		visibilityFilter: visibilityFilter(state. visibilityFilter, action),
		todos: todos(state.todos, action)
	}
}
```

注意每个 reducer 只负责管理全局 state 中它负责的一部分。每个 reducer 的 state 参数都不通，分别对应它管理的那部分 state 数据。

Redux 提供了 `combineReducers()` 工具类来做上面 todos 做的事情，这样能消灭一些样板代码。

```js
import { combineReducers } from 'redux';

const todoApp = combineReducers({
	visibiltiyFilter,
	todos
})

export default todoApp;
```
上面写法和下面完全是等价的。

```js
export default function todoApp(state = {}, action) {
	return {
		visibilityFilter: visibilityFilter(state.visibilityFilter, action),
		todos: todos(state.todos, action)
	}
}
```

`combineReducers()` 所做的只是生成一个函数，这个函数来调用一些列 reducer，每个 reducer 根据它的 key 来筛选出 state 中的一部分数据并处理，然后这个生成的函数再将所有 reducer 的结果合并成一个大的对象。

##### Store

在前面部分，我们学会了使用 action 来描述"发生了什么"，和使用 reducer 来根据 action 更新 state。

Store 就是把它们联系到一起的对象。Store 有以下职责：

* 维持应用的 state
* 提供 getState() 方法获取 state
* 提供 dispatch(action) 方法更新 state
* 通过 subscribe(listener) 注册监听器
* 通过 unsubscribe(listener) 返回的函数注销监听器

根据已有的 reducer 来创建 store 是非常容易的。使用 combineReducers() 将多个 reducer 合并成一个。并将其传递给 createStore()

```js
import { createStore } from 'redux';
import todoApp from './reducers';
let store = createStore(todoApp);
```

createStore() 的第二个参数是可选的，用于设置 state 初始状态。

##### 数据流

严格的单向数据流是 Redux 架构的设计核心。Redux 应用中数据的生命周期遵循下面四个步骤。

##### 1. 调用 store.dispatch(action)

可以在任何地方调用 store.dispatch(action)，action 就是描述发生了什么的普通对象。

##### 2. Redux store 调用传入的 reducer 函数

Store 会把两个参数传入 reducer：当前的 state 和 action。Reducer 是纯函数，它仅仅用于计算下一个 state。

##### 3. 根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树

根 reducer 的结构完全由你决定。Redux 原生提供了 combineReducers() 辅助函数，来根据 reducer 拆分成多个函数，用于分别处理 state 树的一个分支。

##### 4. Redux store 保存了根 reducer 返回的完整的 state 树

这个新的树就是应用的下一个 state！所有订阅 store.dispatch(listener) 的监听器都将被调用

##### 搭配 React

Redux 和 React 之间没有任何关系，Redux 支持 React、Angular、Ember、jQuery 甚至纯 JavaScript。

尽管如此，Redux 还是和 React 这类框架搭配使用，因为这类框架允许你以 state 函数的形式来描述界面，Redux 通过 action 的形式来发起 state 变化。

##### 容器组件和展示组件

Redux 的 React 绑定库是基于容器组件和展示组件相分离的开发思想。

|    |展示组件|容器组件|
|-------------|-------------|-------------|
|作用		|描述如何展现(骨架、样式)|描述如何运行(数据获取，状态更新)|
|直接使用 Redux| 	否|		是|
|数据来源|	props|		监听 Reudx state|
|数据修改|	从 props 调用回调函数|		向 Redux 派发 actions|
|调用方式|	手动| 通常由 React Redux 生成|

大部分的组件都应该是展示型的，但一般需要少数的几个容器组件把它们和 Redux store 连接起来。

展示组件只定义外观并不关心数据来源和如何改变。传入什么就渲染什么。容器组件需要把展示组件连接到 Redux，容器组件就是使用 store.subscribe() 从 Redux state 树中读取部分数据，并通过 props 来把这些数据提供给要渲染的组件。
