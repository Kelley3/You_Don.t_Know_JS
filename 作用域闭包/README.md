
> # 作用域闭包

* [实质问题](#实质问题)
* [现在我懂了](#现在我懂了)
* [循环和闭包](#循环和闭包)
* [重返块作用域](#重返块作用域)
* [模块](#模块)

### 实质问题
* 函数 bar() 的`词法作用域`可以访问 foo() 的`内部作用域`
* bar() 在自己定义的词法作用域`以外`的地方执行
* 拜 bar() 所声明的位置所赐，它拥有涵盖 foo() 内部作用域的闭包，使得该作用域能够`一直存活`
* bar() 依然持有对该作用域的`引用`，而这个引用就叫做`闭包`
* 闭包使得函数可以继续访问`定义时`的词法作用域
* 无论通过何种手段将内部函数传递到所在词法作用域以外，它都会持有对原始定义作用域的引用，`无论在何处执行这个函数都会使用闭包`

```JavaScript
function foo() {
  var a = 2;

  function bar() {
    console.log(a);
  }
  return bar;
}

var baz = foo();
baz();  // 2  -- 朋友，这就是闭包的效果
```

### 现在我懂了
timer 具有涵盖 wait(..) 作用域的闭包
```JavaScript
function wait(message) {
  setTimeout(function timer() {
    console.log(message);
  }, 1000);
}

wait("Hello, closure!");
```

在定时器、事件监听器、Ajax 请求，跨窗口通信、Web Workers 或者任何其他的异步（或者同步）请求中，只要使用了`回调函数`，实际上就是在使用闭包
```JavaScript
function setupBot(name, selector) {
  $(selector).click(function activator() {
    console.log("activating" + name);
  });
}

setupBot("Closure Bot 1", "#bot_1");
setupBot("Closure Bot 2", "#bot_2");
```

`IIFE`严格来讲并不是闭包，因为函数并不是在它本身词法作用域以外执行的，当它的确创建了闭包，并且也是最常用来创建可以被封闭起来的闭包的工具
```JavaScript
var a = 2;

(function IIFE() {
  console.log(a);
})();
```

### 循环和闭包
* `延迟函数的回调会在循环结束时才执行`，所以输出显示的是循环结束时 i 最终值
* 事实上，当定时器运行时即使每一个迭代中执行的时`setTimeout(.., 0)`，所有的回调函数依然是在循环结束后才会被执行
* 尽管五个函数是在各个迭代中分别被定义的，但是它们都被`封闭在一个共享的全局作用域中`，因此实际上只有一个 i
* 将延迟函数的回调`重复定义五次`，完全不使用循环，那它同这段代码是完全等价的

```JavaScript
for(var i = 0; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i);
  }, i*1000);
}
```

用`IIFE`创建作用域
```JavaScript
for(var i = 0; i <= 5; i++) {
  (function IIFE(j) {
    setTimeout(function timer() {
      console.log(j);
    }, j*1000);
  })(i);
}
```

### 重返块作用域
用`let`来劫持块作用域，for 循环头部的 let 每次迭代都会声明，随后的每一个迭代都会使用上一个迭代结束时的值来初始化这个变量
```JavaScript
for(let i = 0; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i);
  }, i*1000);
}
```

### 模块
模块的两个必备条件
* 为创建内部作用域而调用了一个`包装函数`
* 包装函数的返回值必须至少包括一个`内部函数的引用`，这样就会创建涵盖整个包装函数内部作用域的闭包

```JavaScript
function CoolModule() {
  var something = "cool";
  var another = [1, 2, 3];

  function doSomething() {
    console.log(something);
  }
  function doAnother() {
    console.log(another.join("!"));
  }
  return {
    doSomething: doSomething,
    doAnother: doAnother
  };
}

var foo = CoolModule();
foo.doSomething();  // cool
foo.doAnother();    // 1!2!3
```

`单例模式`，且命名作为公共`API`返回的对象
```JavaScript
var foo = (CoolModule(id) {
  function change() {
    // 修改公共 API
    publicAPI.identify = identify2;
  }
  function identify1() {
    console.log(id);
  }
  function identify2() {
    console.log(id.toUpperCase());
  }
  var publicAPI = {
    change: change,
    identify: identify1
  }
  return publicAPI;
})("foo module");

foo.identify();   // foo module
doo.change();
foo.identify2();  // foo MODULE
```
