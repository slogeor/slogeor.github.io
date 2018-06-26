---
title:  React 组件间通信
categories: note
tags: #文章標籤 可以省略
     - react
---

### 父组件向子组件通信

通信是单向的，数据必须是由一方传递到另一方，在 React 中，父组件可以向子组件传递 props 的方法，向子组件进行通信。

如果父组件与子组件之间不止一层级，可以通过 *...运算符*，将父组件的信息，以更简洁的方式传递给更深层级的子组件。

```js
class Parent extends Component {
  render() {
    return <Child_1 msg="hello" />;
  }
}

// 通过 ... 运算符 向 Child_1_1 传递 Parent 组件的信息
class Child_1 extends Component{
  render() {
    return <div>
      <p>{this.props.msg}</p>
      <Child_1_1 {...this.props}/>
    </div>
  }
}

class Child_1_1 extends Component{
  render() {
    return <p>{this.props.msg}</p>
  }
}
```

通过这种方式，父级组件的 props 或 state 的改变，会导致组件本身及其子组件的生命周期的改变。

如果组件嵌套太深，从外到内组件的交流成本会非常高，通过 props 传递值的优势就不会那么明显。建议在开发过程中，减少组件的层次。

### 子组件向父组件通信

父组件向子组件通信可以通过 props 的方法，自顶向下的向子组件进行通信。而子组件向父组件通信，同样也需要父组件向子组件传递 props 进行通信，只是父组件传递的是 `作用域为父组件自身的函数，子组件调用该函数，将子组件想要传递的信息作为参数，传递到父组件的作用域中`。

```js
class Parent extends Component {
  state = {
    msg: 'start'
  };

  transferMsg(msg) {
    this.setState({
      msg
    });
  }

  render() {
    return <div>
        <p>child msg: {this.state.msg}</p>
        <Child_1 transferMsg = {msg => this.transferMsg(msg)} />
      </div>;
  }
}

class Child_1 extends Component{
  componentDidMount() {
    setTimeout(() => {
      this.props.transferMsg('end')
    }, 1000);
  }

  render() {
    return <div>
      <p>child_1 component</p>
    </div>
  }
}
```

![img](https://sfault-image.b0.upaiyun.com/403/745/4037453217-56557a5c39d68_articlex)

### 兄弟组件间的通信

兄弟组件唯一的关联点，就是拥有相同的父组件，参考上面两种关系的通信方法。如果组件 Child_1 向 Child_2 进行通信。可以先通过 Child_1 向 Parent 组件进行通信，然后由 Parent 向 Child_2 组件进行通信。

这个方法存在这样一个问题，由于 Parent 的 state 发生变化，会触发 Parent 及从属于 Parent 的子组件的生命周期。

### content(上下文)

使用上下文，可以让子组件直接访问祖先的数据和函数，无需从祖先组件一层层的传递到子组件中。

```js
class Parent extends Component {
  constructor(props) {
    super(props);
    this.state = {
      curItem: 'item1',
    };

    this.changeItem = this.changeItem.bind(this);
  }

  changeItem(item) {
    this.setState({
      curItem: item
    });
  }

  getChildContext() {
    return {
      curItem: this.state.curItem,
      changeItem: this.changeItem,
    }
  }

  render() {
    return (
      <div>
        <ChildWrapper />
        <ListWrapper changeItem={this.changeItem}/>
      </div>
    )
  }
}

Parent.childContextTypes = {
  curItem: PropTypes.string,
  changeItem: PropTypes.func,
};

const ChildWrapper = ({children}, context) =>
  <button>
    <div>{children}</div>
    {this.context.curItem}
  </button>;

ChildWrapper.contextTypes = {curItem: PropTypes.string};
```

* childContentTypes 是用于验证上下文的数据类型，这个属性是必须要有的，否则会报错
* getChildContent 用于指定子组件可以访问的上下文数据
* 子组件需在 contentTypes 中指定要访问的数据，否则 this.content 访问不到

### 观察者模式

观察者模式也叫做发布者-订阅者模式，发布者发布事情，订阅者监听事件并做出反应。

```js
// eventProxy.js
'use strict';
const eventProxy = {
  onObj: {},
  oneObj: {},
  on: function(key, fn) {
    if(this.onObj[key] === undefined) {
      this.onObj[key] = [];
    }

    this.onObj[key].push(fn);
  },
  one: function(key, fn) {
    if(this.oneObj[key] === undefined) {
      this.oneObj[key] = [];
    }

    this.oneObj[key].push(fn);
  },
  off: function(key) {
    this.onObj[key] = [];
    this.oneObj[key] = [];
  },
  trigger: function() {
    let key, args;
    if(arguments.length == 0) {
      return false;
    }
    key = arguments[0];
    args = [].concat(Array.prototype.slice.call(arguments, 1));

    if(this.onObj[key] !== undefined
      && this.onObj[key].length > 0) {
      for(let i in this.onObj[key]) {
        this.onObj[key][i].apply(null, args);
      }
    }
    if(this.oneObj[key] !== undefined
      && this.oneObj[key].length > 0) {
      for(let i in this.oneObj[key]) {
        this.oneObj[key][i].apply(null, args);
        this.oneObj[key][i] = undefined;
      }
      this.oneObj[key] = [];
    }
  }
};

export default eventProxy;
```

事件绑定和解绑可以分别放在 componentDidMount 和 componentWillUnMount 中。由于事件是全局的，最好保证在 componentWillUnMount 中解绑事件。

事件可以很好的解决组件间通信，但频繁的使用事件实现组件通信会使整个程序的数据流向变得复杂。因此，组件间的沟通尽量还是遵循单向数据流动。

### Redux

Redux 对于组件间的解耦提供了很大的便利，如果你在考虑该不该使用 Redux 的时候，社区里有一句话说，`当你不知道该不该使用 Redux 的时候，那就是不需要的`。

Redux 用起来一时爽，重构或者将项目留给后人的时候，就是个大坑，Redux 中的 dispatch 和 subscribe 方法遍布代码的每一个角落。

Flux 设计中的 Controller-Views 概念就是为了解决这个问题出发的，将所有的 subscribe 都置于 Parent 组件（Controller-Views），由最上层组件控制下层组件的表现，然而，这不就是我们所说的子组件向父组件通信这种方式了。

### 参考文献

* [http://taobaofed.org/blog/2016/11/17/react-components-communication/](http://taobaofed.org/blog/2016/11/17/react-components-communication/)
