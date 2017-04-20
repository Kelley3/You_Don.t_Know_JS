
> # 动态作用域

* 如果 JavaScript 具有`动态作用域`，理论上，下面代码中的 foo() 在执行时将会输出`3`
* 事实上`JavaScript`并不具有动态作用域，它`只有词法作用域`，简单明了，但是`this`机制某种程度上很像动态作用域
* `词法作用域`是在写代码时或者说定义时确定的，而`动态作用域`是在运行时确定的（`this`也是）

```JavaScript
function foo() {
  console.log(a); // 2
}
function bar() {
  var a = 3;
  foo();
}

var a = 2;
bar();
```
