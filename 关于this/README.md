
> # 关于this

* [为什么要用 this](#为什么要用 this)
* [误解](#误解)
* [this 到底是什么](#this 到底是什么)

### 为什么要用 this
this 提供了一种更`优雅`的方式来`隐式”传递“一个对象的引用`，因此可以将 API 设计的更加`简洁`并且`易于复用`
```JavaScript
function identify() {
  return this.name.toUpperCase();
}
function speak() {
  var greeting = "Helo, I'm " + identify.call(this);
  console.log(greeting);
}
var me = {
  name: "Kyle"
};
var you = {
  name: "Reader"
};

console.log(identify.call(me));   // KYLE
console.log(identify.call(you));  // READER
speak.call(me); // Hello, I'm Kyle
speak.call(you); // Hello, I'm Reader
```

### 误解
`this 指向函数自身`
```JavaScript
function foo(num) {
  console.log("foo：" + numm);
  this.count++;
}
foo.count = 0;
var i;
for (i = 0; i < 10; i++) {
  if(i > 5) {
    foo(i);
  }
}
foo: 6
foo: 7
foo: 8
foo: 9

// foo 被调用了多少次
console.log(foo.count); // 0
// correct
// foo.call(foo, i);
```

`this 指向函数的作用域`
```JavaScript
function foo() {
  var a = 2;
  this.bar();
}
function bar() {
  console.log(this.a);
}

foo();  // ReferenceError：a is not defined
```
`使用 this 不可能在词法作用域中查找到什么`，作用域“对象”无法通过 JavaScript 代码访问，它存在于 JavaScript 引擎内部

### this 到底是什么
* 当一个函数被调用时，会创建一个活动记录（`执行上下文`），这个记录包含函数在哪里被调用（`调用栈`），函数的调用方式，传入的参数等信息，this 就是这个记录的一个`属性`，会在函数执行的过程中用到
* this 是在函数被调用时发生的绑定，它指向什么完全`取决于函数在哪里被调用`
