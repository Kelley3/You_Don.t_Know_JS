
> # 函数作用域和块作用域

* [函数中的作用域](#函数中的作用域)
* [隐藏内部实现](#隐藏内部实现)
* [函数作用域](#函数作用域)
* [块作用域](#块作用域)

### 函数中的作用域
属于这个函数的全部变量都可以在整个函数的范围内使用及复用

### 隐藏内部实现
`最小特权原则`：在软件设计中，应该最小限度地暴露必要内容，而将其他内容都“隐藏”起来，比如某个模块或对象的 API 设计
```JavaScript
function doSomething(a) {
  function doSomethingElse(a) {
    return a - 1;
  }
  var b;
  b = a + doSomethingElse(a*2);
  console.log(b*3);
}
doSomething(2); // 15
```

`避免冲突`：使用作用域来“隐藏”内部声明
```JavaScript
function foo() {
  function bar(a) {
    i = 3;  // 修改 for 循环所属作用域中的 i
    console.log(a + i);
  }
  for(var i = 0; i < 10; i++) {
    bar(i*2); // 无限循环
  }
}
foo();
// 方法：var i = 3; || var j = 3;
```

`全局命名空间`

很多第三方库通常会在全局作用域中声明一个`名字足够特别的变量`，通常是一个`对象`，用作库的`命名空间`，所有需要暴露给外界的功能都会成为这个命名空间的`属性`，而不是将自己的标识符暴露在顶级的词法作用域中
```JavaScript
var MyReallyCoolLibrary = {
  awesome: "stuff",
  doSomething: function() {
    // ...
  },
  doAnotherThing: function() {
    // ...
  }
};
```

`模块管理`

使用`模块管理器`，任何库都无需将标识符加入到全局作用域中，而是通过依赖管理器的机制将`库的标识符`显示地`导入`到另外一个`特定的作用域`中

### 函数作用域
`匿名函数表达式的缺点`
* 匿名函数在`栈追踪`中不会显示出有意义的函数名，使得`调试很困难`
* 如果没有函数名，当函数需要引用自身时只能使用已经过期的`arguments.callee`引用，比如在`递归`中
* 匿名函数省略了对于代码`可读性/可理解性`很重要的函数名

`匿名和具名`：始终给函数表达式命名是一个最佳实践
```JavaScript
setTimeout(function timeoutHandler() {  // 哈哈，我有名字了
  console.log("I waited 1 second!");
}, 1000);
```

`立即执行函数表达式`：
* `IIFE`：Immediately Invoked Function Expression
* 形如`(function foo(){ ...})()`，函数被包含在一对()中，成为了一个表达式，在末尾再加上一个()可以立即执行这个函数
* 形如`(function foo(){ ...}())`，另外一种写法，功能一致，`选择哪个全凭个人爱好`

普遍用法之一把函数表达式当作函数调用并`传递参数`进去
```JavaScript
var a = 2;
(function IIFE(global) {
  var a = 3;
  console.log(a);// 3
  console.log(global.a);  // 2
})(window);
console.log(a); // 2
```

普遍用途之一`倒置代码的运行顺序`，将需要运行的函数放在第二位，这种模式在`UMD`项目中被广泛使用
```JavaScript
var a = 2;
(function IIFE(def) {
  def(window);
})(function def(global) {
  var a = 3;
  console.log(a); // 3
  console.log(global.a);  // 2
});
```

### 块作用域
`with`

用 with 从对象中创建出的作用域仅在 with 声明中而非外部作用域中有效

`try/catch`

`ES3`规定 try/catch 的`catch 分句`会创建一个块作用域，其中声明的变量仅在`catch 内部`有效
```JavaScript
try {
  undefined();  // 执行一个非法操作来强制制造一个异常
} catch(err) {
  console.log(err); // 能够正常执行
}
console.log(err); // ReferenceError：err not found
```
同一个作用域中两个或多个catch分句`用同样的标识符名称声明错误变量`时，很多静态检查工具还是会发出`警告`，实际上这`并不是重复定义`

`let`
`ES6`引入 let 关键字，用来在任意`代码块`中声明变量

用 let 进行的声明`不会`在块作用域中进行`提升`
```JavaScript
{
  console.log(bar); // ReferenceError
  let bar = 2;
}
```

`for`循环头部的 let 不仅将 i 绑定到了 for 循环的块中，事实上它将其`重新绑定到了循环的每一个迭代中`
```JavaScript
{
  let j;
  for (j = 0; j < 10; j++) {
    let i = j;  // 每个迭代重新绑定
    console.log(i);
  }
  console.log(j); // 10
}
console.log(j); // ReferenceError
```

`const`
`ES6`引入，同样可以用来创建块作用域变量，但其值时固定的（常量）
