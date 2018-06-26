---
title: 深入 Redux 应用架构
categories: note
tags: #文章標籤 可以省略
     - redux
---

##### Redux 是什么

Redux 参考了 Flux 的设计，但是对 Flux 许多冗余的部分做了简化，同时将 Elm 语言中函数式编程的思想融入其中。

`Redux` 本身指 reudx 这个 npm 包，它提供若干 API 让我们使用 reducer 创建 store，并且能够更新 store 中的数据或获取 store 中最新的状态。

`Redux 应用` 则是使用 redux 这个 npm 包并结合了视图层实现以及其他前端应用必备组件组成的完整的类 Flux 思想的前端应用。

### Redux 三大原则

1.单一数据源

在 Redux 的思想里，一个应用永远只有唯一的数据源。使用单一数据源的好处在于整个应该状态都保存在一个对象中，这样我们随时可以提取整个应用的状态进行持久化。

2.状态是只读的

在 Reudx 中，我们不会自己用代码来定义一个 store，取而代之的是我们定义一个 reducer，它的功能是根据当前触发的 action 对当前应用的状态 state 进行迭代。

Redux 提供的 createStore 方法会根据 reducer 生成 store，最后利用 store.dispatch 方法达到修改状态的目的。

3.状态修改均有纯函数完成

在 Redux 里，可以通过定义 reducer 来确定状态的修改，每一个 reducer 都是纯函数。

##### Redux 核心 API

Redux 的核心是一个 store，这个 store 由 Redux 提供的 createStore(reducers, [initialState])方法生成。

在 Redux 里，负责响应 action 并修改数据的角色是 reducer。reducer 本质上是一个函数，其函数签名为 reducer(previousState, action) => newState。可以看出，reducer 在处理 action 的同时，还需要接受一个 previousState 参数。所以 reducer 的职责就是根据 previousState 和 action 计算出新的 newState。

Redux 中最核心的 API 是 createStore。通过createStore 方法创建的 store 是一个对象，它本身又有四个方法。

* getState(): 获取 store 中当前的状态
* dispatcher(action): 分发一个action，并返回这个 action，这是唯一改变 store 中数据的方法
* subscribe(listener): 注册一个监听者，它在 store 发生变化时被调用
* replaceReducer(nextReducer): 更新当前 store 里的 reducer，一般只会在开发模式中调用该方法

##### 与 React 绑定

react-redux 提供一个组件和一个 API 帮助 Redux 和 React 进行绑定，一个是 React 组件 `<Provier />`，一个是 `connect()`。

* `<Provier />` 接受一个 store 作为 props，它是整个 Redux 应用的顶层组件
* `connect()` 提供了在整个 React 应用的任意组件中获取 store 中数据的功能

##### Redux middleware

middleware 提供了一个分类处理 action 的机会。在 middleware 中，你可以检阅每一个流过的 action，挑选出特定类型的 action 进行相应操作，给你一次改变 action 的机会。

Redux 提供了 applyMiddleware 方法来加载 middleware。

```js
import compose from './compose';

export default function applyMiddleware(...middlewares) {
	return (next) => (reducer, initialState) => {
		let store = next(reducer, initialState)；
		let dispatch = store.dispatch;
		let chain = [];

		var middlewareAPI = {
			getState: stroe.getState,
			dispatch: (action) => dispatch(action),
		};

		chain = middlewares.map(middleware => middleware(middlewareAPI));
		dispatch = componse(...chain)(store.dispatch);

		return {
			...stroe,
			dispatch,
		};
	}
}
```

logger middleware 的实现

```js
export default store => next => action => {
	console.log('dispatch:', action);
	next(action);
	console.log('finish:', action);
}
```

###### middleware 运行原理

* 1.函数式编程思想设计
* 2.给 middleware 分发 store
* 3.组合串联 middleware
* 4.在 middleware 中调用 dispatch

middleware 的设计有点特殊，是一个层层包裹的匿名函数，这其实是函数式编程中的 curring，它是一种使用匿名单参数函数来实现多参数函数的方法。appleMiddleware 会对 logger 这个 middleware 进行层层调用，动态地将 store 和 next 参数赋值。

currying 的 middleware 结构的主要好处：

* 易串联: currying 函数具有延迟执行的特性，通过不断 currying 形成的 middleware 可以积累参数，再配合组合的方式，很容易形成 pipeline 来处理数据流
* 共享 store: 在 appleMiddleware 执行的过程中，store 还是旧的，但是因为闭包的存在，applyMiddleware 完成后，所有的 middleware 内部拿到的 store 是最新且相同的

### Redux 异步流

Flux 没有定义在哪里发异步请求，Redux 是如何解决这个问题的。

##### 使用 middleware 简化异步请求

1.redux-thunk

Thunk 函数实现上就是针对多参数的 currying 以实现对函数的惰性求值。任何函数，只要参数有回调函数，就能写成 Thunk 函数的形式。

```js
fs.readFile(fileName, callback);

var readFileThunk = Thunk(fileName);
readFileThunk(callback);

var Thunk = function(fileName) {
	return function(callback) {
		return fs.readFile(fileName, callback);
	};
};
```

redux-thunk 的源码

```js
function createThunkMiddleware(extraArgument) {
	return ({dispatch, getState}) => next => action=> {
		if (typeof action === 'function') {
			return action(dispatch, getState, extarArgument);
		}

		return next(action);
	}
}
```

当 action 为函数的时候，我们并没有调用 next 或 dispatch 方法，而是返回 action 的调用。这里的 action 就是 Thunk 函数，以达到将 dispatch 和 getState 参数传递到函数内的作用。

2.redux-promise

redux-promise 实现过程非常容易理解，即判断 action 或 action.payload 是否为 promise，如果是，就执行 then，返回的结构再发生一次 dispatch。

### Redux 与 路由

在 React 的生态环境中，React Router 是公认的最优秀的路由解决方案。它提供了与 React 思想十分贴合的声明式的路由系统。我们可以通过 `<Router>、<Route>` 这两个标签以及一系列属性定义整个 React 应用的路由方案。

##### React Router 路由的基本原理

路由的基本原理即是保证 View 和 RUL 同步，而 View 可以看成是资源的一种表现。当用户在 Web 界面中进行操作时，应用会在若干个交互状态中切换，路由则会记录下某些重要的状态。

##### React Router 的特性

在 React Router 中，可以把 Router 组件看成一个方法，location 是参数，返回的结果是 View。

* 声明式的路由
* 嵌套路由及路径匹配
* 支持多种路由切换方式

##### React Router Redux

采用 Redux 架构时，所有的应用状态必须放在一个单一的 store 中管理，路由状态也不例外，这就是 Redux Router Redux 为我们实现的主要功能。

* 将 React Router 与 Redux store 绑定
* 用 Redux 的方式改变路由

##### 容器型组件

容器型组件，意为组件是怎么工作的，更具体一些就是数据是怎么更新的。它不会包含任何 Virtual DOM 的修改或组合，也不会包含组件的样式。

如果映射到 Flux 上，那么容器型组件就是与 store 作绑定的组件。如果映射到 Redux 上，那么容器型组件就是使用 connect 的组件。

##### 展示型组件

展示型组件意为组件是怎么渲染的。它包含了 Virtual DOM 的修改与组合，也可能包含组件的样式。

从布局的角度来看，在 Redux 中，强调了 3 种不同类型的布局组件：Layouts、Views 和 Components。

##### 1.Layouts

Layouts 指的是页面布局组件，描述了页面的基本结构，目的是将主框架与页面主体内容分离。它常常是无状态函数，传入主体内容的 children 属性。Layouts 组件就是设置在最外层 Route 中的 component 里。

```js
const layout = ({children}) => (
	<div className='container'>
		<Headers />
		<div className='content'>
			{children}
		</div>
	</div>
);
```

##### 2.Views

Views 指的是子路由入口组件，描述了子路由入口的基本结构，包含此路由下所有的展示型组件。为了保持组件的纯净，我们在这一层组件中定义了数据和 action 的入口。

```js
class HomeView extends Component {
	render() {
		...todo

		return (
			...todo
		);
	}
}
```

##### 3.Components

Components 就是末级渲染组件，描述了路由以下的子组件。它包含具体的业务逻辑和交互，但所有的数据和 action 都是由 Views 传下来的。这也意味着他们是可以完全脱离数据层而存在的展示型组件。

```js
class Cart textends Components {
	constructor(props) {
		super(props);
		...
	}

	render() {
		...
		return ();
	}
}
```
