
> # this全面解析

* [调用位置](#调用位置)
* [绑定规则](#绑定规则)
* [优先级](#优先级)
* [绑定例外](#绑定例外)

### 调用位置
* this 提供了一种更`优雅`的方式来`隐式”传递“一个对象的引用`，因此可以将 API 设计的更加`简洁`并且`易于复用`
* 使用`开发者工具`设置`断点`可以找到`调用栈`，栈中第`二`个元素就是真正的`调用位置`

```JavaScript
function baz() {
  // 当前调用栈是：baz
  // 因此当前调用位置是全局作用域
  console.log("baz");
  bar();  // bar 的调用位置
}
function bar() {
  // 当前调用栈是：baz -> bar
  // 因此当前调用位置在 baz 中
  console.log("bar");
  foo();  // foo 的调用位置
}
function foo() {
  // 当前调用栈是：baz -> bar -> foo
  // 因此当前调用位置在 bar 中
  console.log("foo");
}

baz();  // baz 的调用位置
```

### 绑定规则
`默认绑定`
* 在非`strict mode`下时，默认绑定绑定到`全局对象`

```JavaScript
function foo() {
  "use strict";
  console.log(this.a);
}
var a = 2;

foo();  // TypeError: Cannot read property 'a' of undefined
```

`隐式绑定`
* 对象属性引用链中只有上一层或者说最后一层在调用位置中起作用

```JavaScript
function foo() {
  console.log(this.a);
}
var obj2 = {
  a: 42,
  foo: foo
};
var obj1 = {
  a: 2,
  obj2: obj2
};

obj1.obj2.foo();  // 42
```

`隐式丢失`
```JavaScript
function foo() {
  console.log(this.a);
}
var obj = {
  a: 2,
  foo: foo
};
var a = "oops, global";   // a 是全局对象的属性

// （一）
// 一个不带任何修饰的函数调用，应用默认绑定
var bar = obj.foo;  // 函数别名
bar();  // oops, global

// （二）
// 参数传递其实就是一种隐式赋值
function doFoo(fn) {
  fn();
}
doFoo(obj.foo); // oops global

// 三
setTimeout(obj.foo, 100); // oops global
```

`显式绑定`
* 使用`call()`或`apply()`方法，它们的第一个参数用于指定 this 的绑定对象
* 如果传入的是一个`原始值`（字符串，布尔，或者数字），这个原始值会被转换成它的`对象形式`（new String(..), new Boolean(..), new Number(..)），这通常被称为`装箱`

```JavaScript
function foo() {
  console.log(this.a);
}
var obj = {
  a: 2
};

foo.call(obj);  // 2
```

`硬绑定`
* 无论如何调用 bar，最终都绑定 obj，这种绑定是一种`显式的强制绑定`，我们称之为硬绑定

```JavaScript
function foo(something) {
  console.log(this.a, something);
  return this.a + something;
}
var obj = {
  a: 2
};
// 简单的辅助绑定函数
function bind(fn, obj) {
  return function() {
    return fn.apply(obj, arguments);
  };
}
var bar = bind(foo, obj);
var b = bar(3); // 2 3
console.log(b); // 5
```

`ES5 提供内置方法 bind()`
* bind(..) 会返回一个硬编码的`新函数`，它会把你指定的参数设置为 this 的上下文并调用原始函数

```JavaScript
function foo(something) {
  console.log(this.a, something);
  return this.a + something;
}
var obj = {
  a: 2
};
var bar = foo.bind(obj);
var b = bar(3); // 2 3
console.log(b); // 5
```

`API 调用”上下文“`
* 第三方库的许多函数，以及 JavaScript 语言和宿主环境中许多新的内置函数，都提供了一个可选的参数，通常被称为`上下文`，其作用和 bind(..) 一样，确保你的回调函数使用指定的 this

```JavaScript
function foo(el) {
  console.log(el, this.id);
}
var obj = {
  id: "awesome"
};

[1, 2, 3].forEach(foo, obj);
// 1 awesome 2 awesome 3 awesome

```

`new 绑定`

在 JavaScript 中，`构造函数`只是一些使用 new 操作符时被调用的`普通函数`，它们并不会属于某个类，也不会实例化一个类，使用 new 来调用函数时，会执行以下操作
* 创建（或者说构造）一个`全新的对象`
* 这个新对象会被执行[[Prototype]]连接
* 这个新对象会`绑定`到函数调用的 this
* 如果函数没有返回其他对象，那么 new 表达式中的函数用会`自动返回这个新对象`

```JavaScript
function foo(a) {
  this.a = a;
}
var bar = new foo(2);
console.log(bar.a); // 2
```

### 优先级
毫无疑问，`默认绑定`的优先级最低

`显示绑定`比`隐式绑定`的优先级`高`
```JavaScript
function foo() {
  console.log(this.a);
}
var obj1 = {
  a: 2,
  foo: foo
};
var obj2 = {
  a: 3,
  foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3
obj1.foo.call(obj2);  // 3
obj2.foo.call(obj1);  // 2
```

`new 绑定`比`隐式绑定`的优先级`高`
```JavaScript
function foo(something) {
  this.a = something;
}
var obj1 = {
  foo: foo
};
var obj2 = {};

obj1.foo(2);
console.log(obj1.a);  // 2
obj1.foo.call(obj2, 3);
console.log(obj2.a);  // 3

var bar = new obj1.foo(4);
console.log(obj1.a);  // 2
console.log(bar.a);   // 4
```

`new 操作符`修改 this 的绑定
```JavaScript
function foo(something) {
  this.a = something;
}
var obj1 = {};
var bar = foo.bind(obj1);
bar(2);
console.log(obj1.a);  // 2

var baz = new bar(3);
console.log(obj1.a);  // 2
console.log(baz.a);   // 3
```

`判断 this`
* 函数是否在`new`中调用（new 绑定）？绑定到`新创建的对象`
* 函数是否通过`call, apply`（显式绑定）或者硬绑定调用？绑定到`指定的对象`
* 函数是否在某个`上下文对象`中调用（隐式绑定）？绑定到那个`上下文对象`
* 如果都不是的话，使用`默认绑定`。如果在严格模式下，绑定到`undefined`，如果不是则绑定到`全局对象`

### 绑定例外
`被忽略的 this`
* 如果你把`null`或者`undefined`作为 this 的绑定对象传入`call, apply`或者`bind`，这些值在调用时会被忽略，实际应用的时`默认绑定`规则
* 这种方式可能会导致许多`难以分析和追踪`的bug

`更安全的 this`
* 在忽略 this 绑定时总是传入一个`DMZ对象`（demilitarized zone），把 this 绑定到这个对象不会对全局对象产生任何影响
* 在 JavaScript 中创建一个空对向最简单的办法是`Object.create(null)`，它与`{}`很像，但是他不会创建`Object.prototype`这个委托，所以它比 {} 更空

```JavaScript
function foo(a, b) {
  console.log("a:" + a + ", b:" + b);
}
// 我们的 DMZ 对象
var DMZ = Object.create(null);

// 把数组“展开”成参数
foo.apply(DMZ, [2, 3]);  // a:2, b:3
// 使用 bind(..) 进行柯里化
var bar = foo.bind(DMZ, 2);
bar(3); // a:2, b:3
```

`间接引用`
* 赋值表达式`p.foo = o.foo`的返回值时目标函数的引用，因此调用位置是`foo`
* 对于`默认绑定`来说，决定 this 绑定对象的并不是`调用位置`是否处于严格模式，而是`函数体`是否处于严格模式

```JavaScript
function foo() {
  console.log(this.a);
}
var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo();  // 3
(p.foo = o.foo)();  // 2
```

`this 词法`
* ES6 中的箭头函数不使用 this 的四种标准规则，而是根据外层（`函数或者全局`）作用域来决定 this，这跟`self = this`机制一样

`箭头函数的词法作用域`
* 箭头函数的绑定无法被修改，new 也不行

```JavaScript
function foo() {
  // 返回一个箭头函数
  return (a) => {
    // this 继承自 foo()
    console.log(this.a);
  }
}
var obj1 = {
  a: 2
};
var obj2 = {
  a: 3
};

var bar = foo.call(obj1);
bar.call(obj2); // 2
```

`箭头函数最常用于回调函数中`
* 例如事件处理器或定时器

```JavaScript
function foo() {
  setTimeout(() => {
    // 这里的 this 在此法上继承自 foo()
    console.log(this.a);
  }, 100);
}
var obj = {
  a: 2
};

foo.call(obj);  // 2
```

`编写 this 风格的代码应当`
* 只使用`词法作用域`并完全抛弃错误 this 风格的代码
* 完全采用 this 风格，在必要时使用`bind(..)`，尽量避免使用`self = this`和`箭头函数`
