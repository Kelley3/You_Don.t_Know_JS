
> # 提升

* [先有鸡还是先有蛋](#先有鸡还是先有蛋)
* [编译器再度来袭](#编译器再度来袭)
* [函数优先](#函数优先)

### 先有鸡还是先有蛋
* JavaScript 引擎将 var a = 2 当作两个声明，一个 var a `定义`声明在`编译`阶段进行，一个 a = 2 `赋值`声明会留在原地等待`执行`阶段
* 无论作用域中的声明出现在什么地方，都将在代码执行前首先进行处理，这个过程成为`提升`

```JavaScript
a = 2;
var a;
console.log(a); // 2
```
```JavaScript
console.log(a); // undefined
var a = 2;
```

### 编译器再度来袭
* 先有`声明`后有`赋值`
* 只有声明本身会被`提升`，而赋值或其他运行逻辑会`留在原地`
* 函数声明会被提升，`函数表达式`不会

```JavaScript
foo();  // TypeError
bar();  // ReferenceError

var foo = function bar() {
  // ...
}
```

### 函数优先
函数会首先被提升，然后才是变量
```JavaScript
foo();  // 1
var foo;

function foo() {
  console.log(1);
}
foo = function() {
  console.log(2);
}
```

尽管重复的 var 声明会被`忽略`掉，但出现在后面的函数声明还是可以`覆盖`前面的
```JavaScript
foo();  // 3

function foo() {
  console.log(1);
}
var foo = function() {
  console.log(2);
}
function foo() {
  console.log(3);
}
```
