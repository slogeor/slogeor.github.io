---
title: 模块化
categories: note
tags: #文章標籤 可以省略
---

### CommonJS

CommonJS 规范加载模块是同步的，也就是说，只有模块加载完成，才能执行后面的操作。

CommonJS 还指出，每个文件就是一个模块，有自己的作用域，在一个文件里定义的变量、函数、类，都是私有的，对外不可见。

```js
// example.js
var x = 5;
var addX = function (val) {
	return val + x;
}
```

` example.js` 的变量 `x` 和 函数 `addX` 都是私有的，对其他文件不可见。

CommonJS 规范规定，每个模块内部，module 变量代表当前模块，这个变量就是一个对象，它的 exports 属性是对外的接口。加载某个模块，其实就是加载该模块的 module.exports 属性。

```js
// example.js
var x = 5;
var addX = function (val) {
	return val + x;
}

module.exports.x = 5;
module.exports.addX = addX;
console.log(module);
```

运行上面代码输出如下：

```js
Module {
  id: '.',
  exports: { x: 5, addX: [Function: addX] },
  parent: null,
  filename: '/Users/xiao.geng/job/github/awesome/test/module/CommonJS/path.js',
  loaded: false,
  children: [],
  paths:
   [ '/Users/xiao.geng/job/github/awesome/test/module/CommonJS/node_modules',
     '/Users/xiao.geng/job/github/awesome/test/module/node_modules',
     '/Users/xiao.geng/job/github/awesome/test/node_modules',
     '/Users/xiao.geng/job/github/awesome/node_modules',
     '/Users/xiao.geng/job/github/node_modules',
     '/Users/xiao.geng/job/node_modules',
     '/Users/xiao.geng/node_modules',
     '/Users/node_modules',
     '/node_modules' ] }
```
**CommonJS 模块的特点**

* 所有代码都运行在模块作用域，不会污染全局作用域
* 模块可以加载多次，但只会在第一次加载时运行一次，然后运行结果就被缓存了
* 模块加载的顺序，就是在代码中出现的顺序

看下面一段代码：

```js
// index.js
var math = require('./example.js');
console.log(math);
console.log(math.addX(1))
math.x = 6;
var math = require('./example.js');
console.log(math);
console.log(math.addX(1))

```

运行 node index.js，结果如下：

```js
{ x: 5, addX: [Function: addX] }
6
{ x: 6, addX: [Function: addX] }
6
```

虽然在 index.js 里对 math.x 进行了赋值，但 addX 方法里引用的 X 还是原来的值。调用 `math.addX(1)` 输出的还是 6。

在 `index.js` 修改 math.x; 虽然 math 对象改变了，但是 addX 方法里的 X 并不会发生改变。还是在引入 example 时，执行完后 X 的值。

在来看一段代码：

```js
// example.js
var a1 = require('./example.js')
console.log('-----第一次输出-------')
console.log(a1)
console.log(a1.addX(1))
console.log('-----第二次输出，修改a1数组的值-------')
a1.a[0] = 8;
console.log(a1)
console.log(a1.addX(1))


module.exports.a = a;
module.exports.addX = addX;

// index.js
var a1 = require('./example.js')
console.log(a1)
console.log(a1.addX(1))
a1[0] = 8;
console.log(a1)
console.log(a1.addX(1))
```

输出结果如下：

```js
-----第一次输出-------
{ a: [ 5 ], addX: [Function: addX], print: [Function: print] }
a: [ 5 ]
6
-----第二次输出，修改a1数组的值-------
{ a: [ 8 ], addX: [Function: addX], print: [Function: print] }
a: [ 8 ]
6
```
虽然 X 的值来源 a[0]，a 是数组类型，但 X 仍然不会更新。我的理解，执行完 example.js 后，再次调用 addX 的时候，X 和 a[0] 之间的关系以及被切断，但是 a 数组却能拿到最新的值[a 是引用类型，外部的修改会影响到内部]，但一般不建议这样使用，因为这样违反了模块的变量和方法的私有性。

##### module 对象

Node 内部提供了一个 Module 构建函数，所有模块都是 Module 的实例。每个模块内部，都有一个 module 对象，代表当前模块。

```js
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  if (parent && parent.children) {
    parent.children.push(this);
  }

  this.filename = null;
  this.loaded = false;
  this.children = [];
}
module.exports = Module;
```

- module.id 模块的识别符，通常是带有绝对路径的模块文件
- module.filename 模块的文件名，带有绝对路径
- module.loaded 返回一个布尔值，表示模块是否已经完成加载
- module.parent 返回一个对象，表示调用该模块的模块
- module.children 返回一个数组，表示该模块要用到的其他模块
- module.exports 表示模块对外输出的值

module.exports 属性表示当前模块对外输出的接口，其他模块加载该模块，实际上就是读取 module.exports 变量。

Node 为每个模块提供一个 exports 变量，指向 module.exports。可以向 exports 对象添加方法，但不能直接将 exports 变量指向一个值，这样会切断 exports 与 module.exports 的联系。

##### node module 加载过程

```js
Module._load = function(request, parent, isMain) {
  if (parent) {
    debug('Module._load REQUEST %s parent: %s', request, parent.id);
  }

  var filename = Module._resolveFilename(request, parent, isMain);

  var cachedModule = Module._cache[filename];
  if (cachedModule) {
    return cachedModule.exports;
  }

  if (NativeModule.nonInternalExists(filename)) {
    debug('load native module %s', request);
    return NativeModule.require(filename);
  }

  var module = new Module(filename, parent);

  if (isMain) {
    process.mainModule = module;
    module.id = '.';
  }

  Module._cache[filename] = module;

  tryModuleLoad(module, filename);

  return module.exports;
};
```

* 1.获取文件的名称及路径
* 2.根据文件名称去缓存(_cache)里读取，读取的是 module 对象
* 3.如果存在缓存，直接返回 module 对象的 exports 属性
* 4.如果不存在缓存，读取文件内容之后，使用 module.compile() 执行文件代码
* 5.加载成功、缓存 module 对象，并返回这个对象的 exports 属性
* 6.更新 module 的属性（id、filename、loaded、exports）

##### 加载规则

根据参数的格式不同， require 命令去不同的路径寻找模块文件。

- 1.参数以 '/' 开头的，则表示加载的是一个位于绝对路径的文件
- 2.参数以 './' 开头的，则表示加载的是一个相对路径的模块文件
- 3.参数不以 './' 或 '/' 开头的，表示加载的是一个默认提供的核心模块
- 4.如果指定的模块文件没有发现，Node 会尝试为文件名添加 .js，.json，.node 后，再去搜索

##### 模块的缓存

所有缓存的模块都保存在 require.cache 之中，如果想删除模块的缓存，可以这样写。

```js
delete require.cache[moduleName];
```

缓存是根据绝对路径识别模块的，如果同样的模块名，但保存的路径不同，require 命令还是会重新加载的。

##### 模块的循环引用

如果发生模块的循环加载，即 A 加载 B，B 又加载 A，则 B 将加载 A 的不完整版本。

先看段代码：

```js
// a.js
exports.x = 'a1';
console.log('a.js:', module.exports);
console.log('--------a1 jump------');
console.log('a.js ', require('./b.js').x);
exports.x = 'a2';
console.log('--------a.js end------');
console.log('a.js:', module.exports);


// b.js
exports.x = 'b1';
console.log('b.js:', module.exports);
console.log('-------b.js jump--------');
console.log('b.js ', require('./a.js').x);
exports.x = 'b2';
console.log('--------b.js end------');
console.log('b.js:', module.exports);


// index.js
console.log('index.js ', require('./a.js').x);
console.log('index.js ', require('./b.js').x);
```

输出结果：

```js
a.js: { x: 'a1' }
--------a1 jump------
b.js: { x: 'b1' }
-------b.js jump--------
b.js  a1
--------b.js end------
b.js: { x: 'b2' }
a.js  b2
--------a.js end------
a.js: { x: 'a2' }
main.js  a2
main.js  b2
```

修改 index.js

```js
console.log('index.js ', require('./a.js').x);
console.log('index.js ', require('./b.js').x);
console.log('---------index-----')
console.log('index.js ', require('./a.js').x);
console.log('index.js ', require('./b.js').x);
```

输出结果：

```js
a.js: { x: 'a1' }
--------a1 jump------
b.js: { x: 'b1' }
-------b.js jump--------
b.js  a1
--------b.js end------
b.js: { x: 'b2' }
a.js  b2
--------a.js end------
a.js: { x: 'a2' }
index.js  a2
index.js  b2
---------index-----
index.js  a2
index.js  b2
```

### ES6

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。

CommonJS 和 AMD 模块都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查询对象属性。

```js
// CommonJS模块
let { stat, exists, readFile } = require('fs');

// 等同于
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
```
上面代码的实质是整体加载 fs 模块，生成一个对象(_fs)，然后再从这个对象上面读取 3 个方法。这种加载成为"运行时加载"，因为只有运行时才能得到这个对象，导致完全没办法在编译时做 "静态优化"。

ES6 模块不是对象，而是通过 export 命令指定输出的代码，再通过 import 命令输入。

```js
// ES6模块
import { stat, exists, readFile } from 'fs';
```

这段代码实质是从 fs 模块加载 3个方法，其他方法不加载。这种加载成为"编译时加载"或静态加载。即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。

##### ES6 模块带来的好处

* 静态分析成为了可能
* 不再需要 UMD 模块格式，将来服务器和浏览器都将会支持 ES6
* 将来浏览器的新 API 就能用模块格式提供，不在必须做成全局变量或者 navigator 对象的属性
* 不再需要对象作为命名空间，未来这些功能可以通过模块提供

##### export 命令

export 命令用于规定模块的对外接口。一个模块就是一个独立的文件，该文件内部的所有变量，外部无法获取。

export 写法：

```js
export var name = 'slogeor';
export var age = 28;

// 等价于
var name = 'slogeor';
var age = 28;
export { name, age };
```

export 命令后面，使用大括号指定所要输出的一组变量。应该优先使用这种写法，这种写法看着更清晰。

```js
function fun () {};
var m = 1;

export {
	m,
	// fun 重命名为 f1
	fun as f1,
}

// 其他写法
export var m = 1;
```

* export 命令规定，对外的接口必须与模块内部的变量建立一一对应关系
* export 输出的变量可以通过 as 进行重命名
* export 语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。
* export 命令只能放在模块顶层位置

```js
export var foo = 'bar';

setTimeout(() => foo = 'baz', 500);
```

上门代码输出变量 foo，值为 bar，500ms 之后变成 baz。

这一点与 CommonJS 规范完全不同。CommonJS 模块输出的是值的缓存，不存在动态更新。

##### import 命令

import命令用于输入其他模块提供的功能。

```js
// main.js
import {firstName as surname, lastName, year} from './profile';

function setName(element) {
  element.textContent = surname + ' ' + lastName;
}
```
* import 命令接受一对大括号，里面指定要从其他模块导入的变量名。大括号里面的变量名，必须与被导入模块对外接口的名称相同
* import 可以使用 as 关键字，对输入的变量重命名
* import 命令具有提升效果，会提升到整个模块的头部，优先执行
* import 是静态执行，所以不能使用表达式和变量
* 如果多次重复执行同一句 import 语句，那么只会执行一次，而不是执行多次
* import 语句会执行所加载的模块，但不会输入任何值

看代码：

```js
// eg1: 重命名
import { name as alias, age } from './person';

// eg2: 相当于执行了 person 模块，但只会执行一次
import './person';
import './person';

// eg3: import 的 Singleton 模式
import { age } from './person';
import { name } from './person';

// 等同于
import { name, age } from './person';
```

##### 模块的整体加载

除了指定加载某个输出值，还可以使用整体加载，即用星号(*)指定一个对象，所有输出值都加载这个对象上面。

```js
// circle.js
export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}

import * as circle from './circle';

console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
```
模块整体加载所在的那个对象，应该是可以静态分析的，所以不允许运行时改变。

##### export default 命令

为了给用户提供放方便，让他们不用阅读文档就能加载模块，这就要用到 export default 命令，为模块指定默认输出。其他模块加载该模块时，import 可以为其指定任意名字。

```js
// export-default.js
export default function () {
  console.log('foo');
}

// import-default.js
import customName from './export-default';
customName(); // 'foo'
```

默认输出和正常输出对比

```js
// 第一组
export default function crs32() {}
import crs32 from 'crs32';

// 第二组
export function crs32() {}
import {crs32} from 'crs32';
```

* 第一组使用 export default 时，对应的 import 语句不需要使用大括号
* 第二组是不使用 export default 时，对应的 import 语句需要使用大括号。


export default 就是输出一个叫做 default 的变量或方法，然后系统允许你它取任意名字

```js
// util.js
function add (x, y) {
	return x + y;
}

export { add as defult };
// 等同于
// export default add;

// index.js
import { default add xxx } from './util';
// 等同于
import xxx from 'util';

```

再看一段代码：

```js
// lodash
export default function (obj) {
  // ···
}

export function each(obj, iterator, context) {
  // ···
}

import _, { each, each as forEach } from 'lodash';
```

模块的接口改名和整体输出

```js
// 接口改变
export { foo as myFoo } from 'my_module';

// 整体输出
export * from 'my_module'
```

##### 模块继承

模块之间也可以有继承。假想有一个 circlePlus 模块，继承了 circle 模块，代码如下：

```js
export * from 'circle';
export var e = 2.71828;
export default function(x) {
	return Math.exp(x);
}
```

`export *` 表示再输出 circle 模块的所有属性和方法，但会忽略 circle 模块的 default 方法。

##### import()

import 的静态加载固然有利于编译器提供效率，但导致无法在运行时加载模块。在语法上，条件加载就不可能实现。然而 `import()` 函数可以完成动态加载。

```js
import(specifier);
```

* import() 和 import 命令的主要区别是 import() 是动态加载
* import() 返回的是一个 pormise 对象
* import() 和 Node 的 require 区别是 import() 是异步加载，require 是同步加载

**使用场景**

* 按需加载
* 条件加载
* 动态的模块路径

##### defer 或 async

浏览器是同步加载 JavaScript 脚本，即渲染引擎遇到 `<script>` 标签就会停下来，等到 执行完脚本，再继续先下渲染。

`<script>` 标签打开 defer 或 async 属性，脚本就会异步加载。渲染引擎遇到这一行命令，就会开始下载外部脚本，但不会等它下载和执行，而是直接加载后面的命令。

* async: 等到整个页面正常渲染结束，才会执行，`下载完就执行`
* defer: 一旦下载完，渲染引擎就会中断执行，执行完在继续渲染，`渲染完在执行`

多个 defer 脚本，是按照引入的顺序的加载。多个 async 脚本是不能保证加载属性的。

##### 加载规则

浏览器加载 ES6 模块，可以使用 `<script>`标签，但要加入 `type="module"` 属性。

```js
<script type="module" src="example.js"></script>
```

浏览器对于带有 type="module" 的 `<script>`，都是异步加载，即等到整个页面渲染完，在执行模块脚本，等同于打开了 `<script>`标签的 defer 属性。

### ES6 模块 与 CommonJS 模块的差异

* 1.CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用
* 2.CommonJS 模块是运行时加载，ES6 模块是编译时输出接口

第一个差异是 CommonJS 模块输出的是值的拷贝，一旦输出一个值，模块内部的变化就不会影响到这个值。

ES6 模块的运行机制与 CommonJS 不一样，JS 引擎对脚本静态分析的时候，遇到模块加载命令 import，就会生成一个只读脚本。等到脚本真正执行时，再跟进这个只读引用，到被加载的模块里去取值。

由于 ES6 输入的模块变量，只是一个符号里链接，所以这个变量是只读的。

export 通过接口，输出的是同一个值，不同脚本加载这个接口，得到是同样的实例。

第二个差异是因为 CommonJS 加载的是一个对象(module.exports属性)，该对象只有在脚本运行完才能生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。

在 ES6 模块之中，顶层的 this 指向 undefined；CommonJS 模块的顶层 this 指向当前模块。

### 循环加载

循环加载指的是 `a.js` 脚本的执行依赖 `b.js` 脚本，而 `b.js`脚本的执行又依赖 `a.js` 脚本。

##### CommonJS

CommonJS 的一个模块，就是一个脚本文件，require 命令第一次加载该脚本，就会执行整个脚本，然后缓存起来，以后需要用这个模块，就会到 exports 属性上取值。

CommonJS 模块无论加载多少次，都只会在第一次加载运行一次，以后在加载，就返回第一次运行的结果，除非手动清除系统缓存。

CommonJS 模块的重要特性就是加载时执行，即脚本代码在 require 的时候，就会全部执行，一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。

**a.js**

```js
exports.done = false;
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 执行完毕');
```

**b.js**

```js
exports.done = false;
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 执行完毕');
```
**main.js**

```js
var a = require('./a.js');
var b = require('./b.js');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);
```

**输出结果**

```js
在 b.js 之中，a.done = false
b.js 执行完毕
在 a.js 之中，b.done = true
a.js 执行完毕
在 main.js 之中, a.done=true, b.done=true
```

CommonJS 输入的是被输出值的拷贝，不是引用。

##### ES6

ES6 模块是动态引用的，如果使用 import 从一个模块加载变量，那些变量不会被缓存，而是成为一个指向被加载模块的引用。

**a.js**

```js
import {bar} from './b.js';
console.log('a.js');
console.log(bar);
export let foo = 'foo';
```

**b.js**

```js
import {foo} from './a.js';
console.log('b.js');
console.log(foo);
export let bar = 'bar';
```

运行 `a.js` 结果如下：

```js
b.js
undefined
a.js
bar
```

再看一个稍微复杂的例子：

```js
// a.js
import {bar} from './b.js';
export function foo() {
  console.log('foo');
  bar();
  console.log('执行完毕');
}
foo();

// b.js
import {foo} from './a.js';
export function bar() {
  console.log('bar');
  if (Math.random() > 0.5) {
    foo();
  }
}
```

按照 CommonJS 规范，上面的代码是没法执行的。但 ES6 可以执行。

```js
$ babel-node a.js
foo
bar
执行完毕

// 执行结果也有可能是
foo
bar
foo
bar
执行完毕
执行完毕
```

原因在于 ES6 加载的变量，都是动态引用其所在的模块，只要引用存在，代码就能执行。

### 总结

* CommonJS 模块是运行时加载，ES6 模块是编译时输出接口
* CommonJS 使用 require 命令加载模块，ES6 使用 import 加载模块
* CommonJS 和 ES6 在处理循环引用上处理方式不一样
* CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用
* CommonJS 加载的是模块的对象的 exports 属性

### 参考资料

* [http://javascript.ruanyifeng.com/nodejs/module.html#toc4](http://javascript.ruanyifeng.com/nodejs/module.html#toc4)
* [Module 的语法](http://es6.ruanyifeng.com/#docs/module)
* [Module 的加载方法](http://es6.ruanyifeng.com/#docs/module-loader#ES6-模块与-CommonJS-模块的差异)
