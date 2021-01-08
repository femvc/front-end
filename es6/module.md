## 目录
---
- [es6 module](#es6-module)
  - [严格模式](#严格模式)
  - [export](#export)
    - [注意](#注意)
      - [动态绑定](#动态绑定)
      - [任意位置](#任意位置)
  - [import](#import)
    - [用as重命名](#用as重命名)
    - [路径可以相对或绝对或者省略](#路径可以相对或绝对或者省略)
    - [不能使用表达式](#不能使用表达式)
    - [只执行加载模块](#只执行加载模块)
    - [多次执行只处理一次](#多次执行只处理一次)
    - [单例形式](#单例形式)
    - [整体加载 import * as ...](#整体加载-import-*-as-)
  - [export default](#export-default)
    - [只能使用一次（但是可以和普通的export同时使用）](#只能使用一次但是可以和普通的export同时使用)
    - [as default 也行](#as-default-也行)
    - [export default a 相当于赋值操作](#export-default-a-相当于赋值操作)
    - [也可以输出类](#也可以输出类)
    - [例子](#例子)
  - [export 和 import 结合](#export-和-import-结合)
    - [改名和整体输出](#改名和整体输出)
    - [导出默认接口](#导出默认接口)
    - [未解决的方法](#未解决的方法)
  - [模块继承](#模块继承)
  - [跨模块常量](#跨模块常量)
    - [专门文件夹做加载文件夹（写好index.js）](#专门文件夹做加载文件夹写好indexjs)
  - [import 无法实现动态加载](#import-无法实现动态加载)
    - [import()](#import)
    - [适用场合](#适用场合)
      - [**按需加载**](#**按需加载**)
      - [条件加载](#条件加载)
      - [动态的模块路径](#动态的模块路径)
    - [注意点](#注意点)
---

# es6 module

ES6 模块的设计思想，**是尽量的静态化**，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

```
// CommonJS模块
let { stat, exists, readFile } = require('fs');
// 等同于
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
```

上面代码的实质是整体加载`fs`模块（即加载`fs`的所有方法），生成一个对象（`_fs`），然后再从这个对象上面读取3个方法。这种加载称为“**运行时加载**”，因为只有运行时才能得到这个对象，**导致完全没办法在编译时做“静态优化”**。

ES6 模块不是对象，而是通过`export`命令显式指定输出的代码，再通过`import`命令输入。

```
// ES6模块
import { stat, exists, readFile } from 'fs';
```

上面代码的实质是从`fs`模块加载3个方法，其他方法不加载。这种加载称为“**编译时加载**”或者**静态加载**，即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。当然，**这也导致了没法引用 ES6 模块本身，因为它不是对象**。

由于 ES6 模块是编译时加载，使得静态分析成为可能。有了它，就能进一步拓宽 JavaScript 的语法，比如引入宏（macro）和类型检验（type system）这些只能靠静态分析实现的功能。

除了静态加载带来的各种好处，ES6 模块还有以下好处。

- 不再需要`UMD`模块格式了，将来服务器和浏览器都会支持 ES6 模块格式。目前，通过各种工具库，其实已经做到了这一点。

- 将来浏览器的新 API 就能用模块格式提供，不再必须做成全局变量或者`navigator`对象的属性。

- 不再需要对象作为命名空间（比如`Math`对象），未来这些功能可以通过模块提供。

## 严格模式

ES6 的模块自动采用严格模式，不管你有没有在模块头部加上`"use strict";`。

严格模式主要有以下限制。

- 变量必须**声明后再使用**

- 函数的参数**不能有同名属性**，否则报错

- 不能使用`with`语句

- 不能对只读属性赋值，否则报错

- **不能使用前缀0表示八进制数**，否则报错

- **不能删除不可删除的属性**，否则报错

- 不能删除变量`delete prop`，会报错，只能删除属性`delete global[prop]`

- `eval`不会在它的外层作用域引入变量

- `eval`和`arguments`不能被重新赋值

- `arguments`**不会自动反映函数参数的变化**

- 不能使用`arguments.callee`

- 不能使用`arguments.caller`

- 禁止`this`指向全局对象

- 不能使用`fn.caller`和`fn.arguments`获取函数调用的堆栈

- 增加了保留字（比如`protected`、`static`和`interface`）

其中，尤其需要注意`this`的限制。ES6 模块之中，顶层的`this`指向`undefined`，即不应该在顶层代码使用`this`。

## export

export可以导出变量，对象，函数……

```javascript
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;
function v1() {...}
export {firstName, lastName, year, v1 };
```

常情况下，`export`输出的变量就是本来的名字，但是可以使用`as`关键字重命名。

```
function v1() { ... }
function v2() { ... }
export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
```

### 注意

#### 一一对应

需要特别注意的是，`export`命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。

```javascript
// 报错
export 1;
// 报错
var m = 1;
export m;
```

上面两种写法都会报错，因为没有提供对外的接口。第一种写法直接输出1，第二种写法通过变量`m`，还是直接输出1。`1`只是一个值，不是接口。正确的写法是下面这样。

```javascript
// 写法一
export var m = 1;
// 写法二
var m = 1;
export {m};   //这里相当export {m:m}
// 写法三
var n = 1;
export {n as m};
```

上面三种写法都是正确的，规定了对外的接口`m`。其他脚本可以通过这个接口，取到值`1`。它们的实质是，在接口名与模块内部变量之间，建立了一一对应的关系。

#### 动态绑定

另外，`export`语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。

```
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
```

上面代码输出变量`foo`，值为`bar`，500毫秒之后变成`baz`。

这一点与 CommonJS 规范完全不同。CommonJS 模块输出的是值的缓存，不存在动态更新，详见下文《Module 的加载实现》一节。

#### 任意位置

最后，`export`命令可以出现在模块的任何位置，**只要处于模块顶层就可以**。如果处于块级作用域内，就会报错，下一节的`import`命令也是如此。这是因为处于条件代码块之中，就没法做静态优化了，违背了ES6模块的设计初衷。

```
function foo() {
  export default 'bar' // SyntaxError
foo()
```

上面代码中，`export`语句放在函数之中，结果报错。

## import

### 与对外接口名一致

`import`命令接受一对大括号，里面指定要从其他模块导入的变量名。大括号里面的变量名，必须与被导入模块（`profile.js`）**对外接口的名称相同**。

### 用as重命名

如果想为输入的变量重新取一个名字，`import`命令要使用`as`关键字，将输入的变量重命名。

```
import { lastName as surname } from './profile';
```

### 路径可以相对或绝对或者省略

`import`后面的`from`指定模块文件的位置，可以是相对路径，也可以是绝对路径，`.js`路径可以省略。如果只是模块名，**不带有路径，那么必须有配置文件**，告诉 JavaScript 引擎该模块的位置。

```
import {myMethod} from 'util';
```

### 不能使用表达式

由于`import`是静态执行，**所以不能使用表达式和变量**，这些只有在运行时才能得到结果的语法结构。

```
// 报错
import { 'f' + 'oo' } from 'my_module';
// 报错
let module = 'my_module';
import { foo } from module;
// 报错
if (x === 1) {
  import { foo } from 'module1';
} else {
  import { foo } from 'module2';
```

### 只执行加载模块

最后，`import`语句会执行所加载的模块，因此可以有下面的写法。

```
import 'lodash';
```

上面代码仅仅执行`lodash`模块，但是不输入任何值。

### 多次执行只处理一次

多次重复执行同一句`import`语句，那么只会执行一次，而不会执行多次。

```
import 'lodash';
import 'lodash';
```

上面代码加载了两次`lodash`，但是只会执行一次。

### 单例形式

```javascript
import { foo } from 'my_module';
import { bar } from 'my_module';
// 等同于
import { foo, bar } from 'my_module';
```

上面代码中，虽然`foo`和`bar`在两个语句中加载，但是它们对应的是同一个`my_module`实例。也就是说，`import`语句是 Singleton 模式。

### 整体加载 import * as ...

**注意：import * 会忽略default方法**

```javascript
import { area, circumference } from './circle';
console.log('圆面积：' + area(4));
console.log('圆周长：' + circumference(14));
```

上面写法是逐一指定要加载的方法，整体加载的写法如下。

```javascript
import * as circle from './circle';
console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
```

注意，模块整体加载所在的那个对象（上例是`circle`），应该是可以静态分析的，**所以不允许运行时改变。下面的写法都是不允许的**。

```javascript
import * as circle from './circle';
// 下面两行都是不允许的
circle.foo = 'hello';
circle.area = function () {};
```

## export default

### 不需要大括号，不需要指定接口名称

从前面的例子可以看出，使用`import`命令的时候，用户需要知道所要加载的变量名或函数名，否则无法加载。为了给用户提供方便，让他们不用阅读文档就能加载模块，就要用到`export default`命令，为模块指定默认输出。

```javascript
// export-default.js
export default function () {
  console.log('foo');
```

上面代码是一个模块文件`export-default.js`，它的默认输出是一个函数。

其他模块加载该模块时，`import`命令可以为**该匿名函数指定任意名字**。**(不需要大括号！！！！)**

```javascript
// import-default.js
import customName from './export-default';  
customName(); // 'foo'
```

`export default `命令用在非匿名函数前，也是可以的。

```javascript
// export-default.js
export default function foo() {
  console.log('foo');
// 或者写成
function foo() {
  console.log('foo');
export default foo;
```

上面代码中，`foo`函数的函数名`foo`，在模块外部是无效的。**加载的时候，视同匿名函数加载**。

### 只能使用一次（但是可以和普通的export同时使用）

`export default`命令用于指定模块的默认输出。显然，一个模块只能有一个默认输出，因此`export default`命令只能使用一次。所以，`import`命令后面才不用加大括号，因为只可能对应一个方法。

如果想在一条`import`语句中，同时输入默认方法和其他变量，可以写成下面这样。

```javascript
import _, { each } from 'lodash';
```

对应上面代码的`export`语句如下。

```javascript
export default function (obj) {
  // ···
export function each(obj, iterator, context) {
  // ···
export { each as forEach };
```

上面代码的最后一行的意思是，暴露出`forEach`接口，默认指向`each`接口，即`forEach`和`each`**指向同一个方法**。

### as default 也行

本质上，`export default`就是输出一个叫做`default`的变量或方法，然后系统允许你为它取任意名字。所以，下面的写法是有效的。

```javascript
// modules.js
function add(x, y) {
  return x * y;
export {add as default};
// 等同于
// export default add;
// app.js
import { default as xxx } from 'modules';
// 等同于
// import xxx from 'modules';
```

### export default a 相当于赋值操作

正是因为`export default`命令其实只是输出一个叫做`default`的变量，**所以它后面不能跟变量声明语句**。

```javascript
// 正确
export var a = 1;
// 正确
var a = 1;
export default a;
// 错误
export default var a = 1;
```

上面代码中，`export default a`的含义是将变量`a`的**值赋给**变量`default`。所以，最后一种写法会报错。

同样地，因为`export default`本质是将该命令后面的值，赋给`default`变量以后再默认，所以直接将一个值写在`export default`之后。

```javascript
// 正确
export default 42;
// 报错
export 42;
```

上面代码中，后一句报错是因为没有指定对外的接口，而前一句指定外对接口为`default`。

### 也可以输出类

`export default`也可以用来输出类。

```javascript
// MyClass.js
export default class { ... }
// main.js
import MyClass from 'MyClass';
let o = new MyClass();
```

### 例子

有了`export default`命令，输入模块时就非常直观了，以输入 lodash 模块为例。

```javascript
import _ from 'lodash';
```

## export 和 import 结合

### export ... from

如果在一个模块之中，先输入后输出同一个模块，`import`语句可以与`export`语句写在一起。

```javascript
export { foo, bar } from 'my_module';
// 等同于
import { foo, bar } from 'my_module';
export { foo, bar };
```

### 改名和整体输出

模块的接口改名和整体输出，也可以采用这种写法。

```javascript
// 接口改名
export { foo as myFoo } from 'my_module';
// 整体输出
export * from 'my_module';
```

### 导出默认接口

默认接口的写法如下。

```javascript
export { default } from 'foo';
```

**具名接口改为默认接口**的写法如下。

```javascript
export { es6 as default } from './someModule';
// 等同于
import { es6 } from './someModule';
export default es6;
```

同样地，**默认接口也可以改名为具名接口**。

```javascript
export { default as es6 } from './someModule';
```

### 未解决的方法

下面三种`import`语句，没有对应的复合写法。

```javascript
import * as someIdentifier from "someModule";
import someIdentifier from "someModule";
import someIdentifier, { namedIdentifier } from "someModule";
```

为了做到形式的对称，现在有[提案](https://github.com/leebyron/ecmascript-export-default-from)，提出补上这三种复合写法。

```javascript
export * as someIdentifier from "someModule";
export someIdentifier from "someModule";
export someIdentifier, { namedIdentifier } from "someModule";
```

## 模块继承

### export * from ...

假设有一个`circleplus`模块，继承了`circle`模块。

```javascript
// circleplus.js
export * from 'circle';    // 先输出父模块所有的方法
export var e = 2.71828182846; // 添加自己的方法
export default function(x) {
  return Math.exp(x);
```

注意，`export *`命令会忽略`circle`模块的`default`方法。然后，上面代码又输出了自定义的`e`变量和默认方法。这时，也可以将`circle`的属性或方法，改名后再输出。

```javascript
// circleplus.js
export { area as circleArea } from 'circle';
```

上面代码表示，只输出`circle`模块的`area`方法，且将其改名为`circleArea`。

加载上面模块的写法如下。

```javascript
// main.js
import * as math from 'circleplus';
import exp from 'circleplus';
console.log(exp(math.e));
```

上面代码中的`import exp`表示，将`circleplus`模块的默认方法加载为`exp`方法。

## 跨模块常量

本书介绍`const`命令的时候说过，`const`声明的常量只在当前代码块有效。如果想设置跨模块的常量（即跨多个文件），或者说一个值要被多个模块共享，可以采用下面的写法。

```javascript
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;
// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3
// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```

### 专门文件夹做加载文件夹（写好index.js）

如果要使用的常量非常多，可以建一个专门的`constants`目录，将各种常量写在不同的文件里面，保存在该目录下。

```javascript
// constants/db.js
export const db = {
  url: 'http://my.couchdbserver.local:5984',
  admin_username: 'admin',
  admin_password: 'admin password'
};
// constants/user.js
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator'];
```

然后，将这些文件输出的常量，合并在`index.js`里面。

```javascript
// constants/index.js
export {db} from './db';
export {users} from './users';
```

使用的时候，直接加载`index.js`就可以了。（可以不用制定index.js，会默认先找这个，查询规则：1. 同名js，2. 同名文件夹中的index.js）

```javascript
// script.js
import {db, users} from './constants';
```

## import 无法实现动态加载

前面介绍过，`import`命令会被 JavaScript 引擎静态分析，先于模块内的其他模块执行（叫做”连接“更合适）。所以，下面的代码会报错。

```javascript
// 报错
if (x === 2) {
  import MyModual from './myModual';
```

这样的设计，固然有利于编译器提高效率，**但也导致无法在运行时加载模块**。从语法上，条件加载就不可能实现。如果`import`命令要取代 Node 的`require`方法，这就形成了一个障碍。因为`require`是运行时加载模块，`import`命令无法取代`require`的动态加载功能。

```javascript
const path = './' + fileName;
const myModual = require(path);
```

上面的语句就是动态加载，`require`到底加载哪一个模块，只有运行时才知道。`import`语句做不到这一点。

### import()

因此，有一个[提案](https://github.com/tc39/proposal-dynamic-import)，建议引入`import()`函数，完成动态加载。

```javascript
import(specifier)
```

上面代码中，`import`函数的参数`specifier`，指定所要加载的模块的位置。`import`命令能够接受什么参数，`import()`函数就能接受什么参数，两者区别主要是后者为动态加载。

`import()`返回一个 Promise 对象。下面是一个例子。

```javascript
const main = document.querySelector('main');
import(`./section-modules/${someVariable}.js`)
  .then(module => {
    module.loadPageInto(main);
  })
  .catch(err => {
    main.textContent = err.message;
  });
```

`import()`函数可以用在任何地方，**不仅仅是模块，非模块的脚本也可以使用**。它是运行时执行，也就是说，什么时候运行到这一句，也会加载指定的模块。另外，`import()`函数与所加载的模块没有静态连接关系，这点也是与`import`语句不相同。

`import()`类似于 Node 的`require`方法，**区别主要是前者是异步加载，后者是同步加载**。

### 适用场合

下面是`import()`的一些适用场合。

#### **按需加载**

`import()`可以在需要的时候，再加载某个模块。

```javascript
button.addEventListener('click', event => {
  import('./dialogBox.js')
  .then(dialogBox => {
    dialogBox.open();
  })
  .catch(error => {
    /* Error handling */
  })
});
```

上面代码中，`import()`方法放在`click`事件的监听函数之中，只有用户点击了按钮，才会加载这个模块。

#### 条件加载

`import()`可以放在`if`代码块，根据不同的情况，加载不同的模块。

```javascript
if (condition) {
  import('moduleA').then(...);
} else {
  import('moduleB').then(...);
```

上面代码中，如果满足条件，就加载模块 A，否则加载模块 B。

#### 动态的模块路径

`import()`允许模块路径动态生成。

```javascript
import(f())
.then(...);
```

上面代码中，根据函数`f`的返回结果，加载不同的模块。

### 注意点

`import()`加载模块成功以后，这个模块会作为一个对象，当作`then`方法的参数。因此，可以使用对象解构赋值的语法，获取输出接口。

```javascript
import('./myModule.js')
.then(({export1, export2}) => {
  // ...·
});
```

上面代码中，`export1`和`export2`都是`myModule.js`的输出接口，可以解构获得。

如果模块有`default`输出接口，可以用参数直接获得。

```javascript
import('./myModule.js')
.then(myModule => {
  console.log(myModule.default);
});
```

上面的代码也可以使用具名输入的形式。

```javascript
import('./myModule.js')
.then(({default: theDefault}) => {
  console.log(theDefault);
});
```

如果想同时加载多个模块，可以采用下面的写法。

```javascript
Promise.all([
  import('./module1.js'),
  import('./module2.js'),
  import('./module3.js'),
])
.then(([module1, module2, module3]) => {
   ···
});
```

`import()`也可以用在 async 函数之中。

```javascript
async function main() {
  const myModule = await import('./myModule.js');
  const {export1, export2} = await import('./myModule.js');
  const [module1, module2, module3] =
    await Promise.all([
      import('./module1.js'),
      import('./module2.js'),
      import('./module3.js'),
    ]);
main();
```




# ECMAScript Proposal: export default from

**Stage:** 1

**Author:** Lee Byron

**Reviewers:** Caridy Patiño

**Specification:** https://tc39.github.io/proposal-export-default-from/

**AST:** [ESTree.md](./ESTree.md)

**Transpiler:** See Babel's [export-default-from](https://babeljs.io/docs/en/babel-plugin-proposal-export-default-from) plugin.

> NOTE: Closely related to the [export-ns-from](https://github.com/tc39/proposal-export-ns-from) proposal.

## Problem statement and rationale

The `export ___ from "module"` statements are a very useful mechanism for
building up "package" modules in a declarative way. In the ECMAScript 2015 spec,
we can:

* export through a single export with `export {x} from "mod"`
* ...optionally renaming it with `export {x as v} from "mod"`.
* We can also spread all exports with `export * from "mod"`.

These three export-from statements are easy to understand if you understand the
semantics of the similar looking import statements.

However there is an import statement which does not have corresponding
export-from statement, exporting the ModuleNameSpace object as a named export.

Example:

```js
export someIdentifier from "someModule";
export someIdentifier, { namedIdentifier } from "someModule";
```


### Current ECMAScript 2015 Modules:

Import Statement Form         | [[ModuleRequest]] | [[ImportName]] | [[LocalName]]
---------------------         | ----------------- | -------------- | -------------
`import v from "mod";`        | `"mod"`           | `"default"`    | `"v"`
`import * as ns from "mod";`  | `"mod"`           | `"*"`          | `"ns"`
`import {x} from "mod";`      | `"mod"`           | `"x"`          | `"x"`
`import {x as v} from "mod";` | `"mod"`           | `"x"`          | `"v"`
`import "mod";`               |                   |                |


Export Statement Form           | [[ModuleRequest]] | [[ImportName]] | [[LocalName]] | [[ExportName]]
---------------------           | ----------------- | -------------- | ------------- | --------------
`export var v;`                 | **null**          | **null**       | `"v"`         | `"v"`
`export default function f(){};`| **null**          | **null**       | `"f"`         | `"default"`
`export default function(){};`  | **null**          | **null**       | `"*default*"` | `"default"`
`export default 42;`            | **null**          | **null**       | `"*default*"` | `"default"`
`export {x}`;                   | **null**          | **null**       | `"x"`         | `"x"`
`export {x as v}`;              | **null**          | **null**       | `"x"`         | `"v"`
`export {x} from "mod"`;        | `"mod"`           | `"x"`          | **null**      | `"x"`
`export {x as v} from "mod"`;   | `"mod"`           | `"x"`          | **null**      | `"v"`
`export * from "mod"`;          | `"mod"`           | `"*"`          | **null**      | **null**


### Proposed addition:

Export Statement Form   | [[ModuleRequest]] | [[ImportName]] | [[LocalName]] | [[ExportName]]
---------------------   | ----------------- | -------------- | ------------- | --------------
`export v from "mod";`  | `"mod"`           | `"default"`    | **null**      | `"v"`


## Symmetry between import and export

There's a syntactic symmetry between the export-from statements and the import
statements they resemble. There is also a semantic symmetry; where import
creates a locally named binding, export-from creates an export entry.

As an existing example:

```js
import {v} from "mod";
```

If then `v` should be exported, this can be followed by an `export`. However, if
`v` is unused in the local scope, then it has introduced a name to the local
scope unnecessarily.

```js
import {v} from "mod";
export {v};
```

A single "export from" line directly creates an export entry, and does not alter
the local scope. It is *symmetric* to the similar "import from" statement.

```js
export {v} from "mod";
```

This presents a developer experience where it is expected that replacing the
word `import` with `export` will always provide this symmetric behavior.

However, when there is a gap in this symmetry, it can lead to confusing behavior:

> "I would like to chime in with use-case evidence. When I began using ES6 (via
> babel) and discovered imports could be re-exported I assumed the feature set
> would be symmetrical only to find out quite surprisingly that it was not. I
> would love to see this spec round out what I believe is a slightly incomplete
> feature set in ES6."
>
> - @jasonkuhrt
>
> "I also bumped into this and even thought this was a bug in Babel."
>
> - @gaearon

### Proposed addition:

The proposed addition follows this same symmetric pattern:

Importing the "default" name (existing):

```js
import v from "mod";
```

Exporting that name (existing):

```js
import v from "mod";
export {v};
```

Symmetric "export from" (proposed):

```js
export v from "mod";
```

### Compound "export from" statements:

This proposal also includes the symmetric export-from for the compound imports:

Importing the "default" name as well as named export (existing):

```js
import v, { x, y as w } from "mod";
```

Exporting those names (existing):

```js
import v, { x, y as w } from "mod";
export { v, x, w };
```

Symmetric "export from" (proposed):

```js
export v, {x, y as w} from "mod";
```

> Note: The compound form must also support [export-ns-from](https://github.com/leebyron/ecmascript-export-ns-from) should both proposals be accepted:
>
> ```js
> export v, * as ns from "mod";
> ```

### Exporting a default as default:

One use case is to take the default export of an inner module and export it as
the default export of the outer module. This can be written as:

```js
export default from "mod";
```

This is symmmetric to the (existing) import form:

```js
import default from "mod";
export { default };
```

This is *not additional syntax* above what's already proposed. In fact, this is
just the `export v from "mod"` syntax where the export name happens to be
`default`. This nicely mirrors the other `export default ____` forms in both
syntax and semantics without requiring additional specification. An alteration
to an existing lookahead restriction is necessary for supporting this case.


### Common concerns:

> Do we need this even through you can already do this with `export { default } from "mod"`?

Yes! It is true that you can already "export from" a module's default export
without altering the local scope:

```js
export { default } from "mod";
```

You can even rename the default:

```js
export { default as someIdentifier } from "mod"
```

There is a benefit to this, it's explicit and it is symmetric with the
"import" form:

```js
import { default as someIdentifier } from "mod"
```

However the purpose of this proposal is not to enable a new use case, the
purpose is to provide an expected syntactic form which currently does not exist,
and which favors the "default" export.

That "import" example is not often typed, as there is a preferred shorter syntax:

```js
import someIdentifier from "mod"
```

This proposal argues that a similar shorter syntax should also exist for a
symmetric "export from" form:

```js
export someIdentifier from "mod"
```


### Table showing symmetry

Using the terminology of [Table 40][] and [Table 42][] in ECMAScript 2015, the
export-from form can be created from the symmetric import form by setting
export-from's **[[ExportName]]** to import's **[[LocalName]]** and export-from's
**[[LocalName]]** to **null**.

[Table 40]: http://www.ecma-international.org/ecma-262/6.0/#table-40
[Table 42]: http://www.ecma-international.org/ecma-262/6.0/#table-42

Statement Form                          | [[ModuleRequest]] | [[ImportName]] | [[LocalName]]  | [[ExportName]]
--------------                          | ----------------- | -------------- | -------------- | --------------
`import v from "mod";`                  | `"mod"`           | `"default"`    | `"v"`          |
<ins>`export v from "mod";`</ins>       | `"mod"`           | `"default"`    | **null**       | `"v"`
`import {x} from "mod";`                | `"mod"`           | `"x"`          | `"x"`          |
`export {x} from "mod";`                | `"mod"`           | `"x"`          | **null**       | `"x"`
`import {x as v} from "mod";`           | `"mod"`           | `"x"`          | `"v"`          |
`export {x as v} from "mod";`           | `"mod"`           | `"x"`          | **null**       | `"v"`
`import * as ns from "mod";`            | `"mod"`           | `"*"`          | `"ns"`         |
`export * from "mod";`                  | `"mod"`           | `"*"`          | **null**       | **null** (many)


