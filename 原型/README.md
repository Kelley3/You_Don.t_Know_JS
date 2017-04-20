
> # 原型

* [Prototype](#Prototype)
* [类](#类)
* [原型继承](#原型继承)
* [对象关联](#对象关联)

### Prototype
* 如果要访问对象中并不存在的一个属性，`[[Get]]`操作就会查找对象内部`[[Prototype]]`关联的对象
* 这个关联关系实际上定义了一条`原型链`（有点像嵌套的作用域），在查找属性时会对它进行遍历
* 所有普通对象都有`Object.prototype`，指向原型链的`顶端`，如果在原型链中找不到指定的属性就会停止
* `toString(), valueOf()`和其他一些通用的功能都存在于`Object.prototype`对象上，因此语言中所有的对象都可以使用它们

```JavaScript
var anotherObj = {
  a: 2
};
// 创建一个关联到 anotherObj 的对象
var myObject = Object.create(anotherObj);

for (var k in myObject) {
  console.log("found：" + k);  // found：a
}
console.log("a" in myObject); // true
```

`属性设置和屏蔽`
* 如果 myObject 本身包含 foo，则`修改`已有的属性值
* 如果 foo 不是直接存在于 myObject 中，`[[Prototype]]`链就会被遍历，类似 [[Get]] 操作，如果原型链上找不到 foo，foo 就会`直接被添加`到 myObject
* 如果 foo 存在于原型链上层
  * 且 foo 为`普通数据访问属性`，没有被标记为只读，那就会在 myObject 中添加一个名为 foo 的新属性，它是`屏蔽属性`
  * 且标记为`只读`，在严格模式下，代码会`报错`，在非严格模式下，代码会`被忽略`
  * 并且它是一个`setter`，那就一定会调用这个 setter
* 如果 foo 既存在于 myObject 中，也出现在 [[Prototype]] 链上层，那么就会发生`屏蔽`，myObject.foo 总是选择原型链中最底层的 foo 属性

`隐式产生屏蔽`
* myObject.a++ 相当于 myObject.a = myObject.a + 1
* 如果想让 anotherObject.a 的值增加，唯一的办法是 anotherObject.a++

```JavaScript
var anotherObject = {
  a: 2
};
var myObject = Object.create(anotherObject);
console.log(anotherObject.a);   // 2
console.log(myObject.a);        // 2
console.log(anotherObject.hasOwnProperty("a")); // true
console.log(myObject.hasOwnProperty("a"));      // false
myObject.a++; // 隐式屏蔽
console.log(anotherObject.a); // 2
console.log(myObject.a);      // 3
console.log(myObject.hasOwnProperty("a"));  // true
```

### 类
* `JavaScript 中只有对象`
* 所有的函数默认都会拥有一个名为`prototype`的共有并且不可枚举的属性，它会指向另一个对象，即该函数的`原型`
* 关联两个对象最常用的办法是使用`new`关键词进行函数调用，在调用的4个步骤中会`创建一个关联其他对象的新对象`
* 使用`new`调用函数时会把新对象的`.prototype`属性关联到`其他对象`
* 带`new`的函数调用通常被称为`构造函数调用`，尽管他们实际上和传统面向类语言中的`类构造函数`不一样
* 虽然这些 JavaScript 机制和传统面向类语言的`类初始化`和`类继承`很相似，但 JavaScript 中的机制有一个核心区别，那就是`不会进行复制`，对象之间是通过`[[prototype]]`链关联的
* `委托`更加准确地描述 JavaScript 中对象的关联机制
* Foo.prototype 默认有一个共有并且不可枚举的属性`.constructor`，这个属性引用的是对象关联的函数
* `.constructor`引用同样被委托给了`Foo.prototype`
* 如果你创建了一个新对象并替换了函数默认的`.prototype`对象引用，那么新对象并不会自动获得`.constructor`属性

```JavaScript
function Foo(name) {
  this.name = name;
}
Foo.prototype.myName = function() {
  return this.name;
};
var a = new Foo("a");
var b = new Foo("b");

console.log(a.myName());  // a
console.log(b.myName());  // b

console.log(a.constructor === Foo);   // true
Foo.prototype = {};  // 创建一个新原型对象
var c = new Foo();
console.log(c.constructor === Foo);     // false
console.log(c.constructor === Object);  // true
```

### 原型继承
* `Bar.prototype = Foo.prototype`只是让 Bar.prototype 直接引用 Foo.prototype，这和我们想要的机制不一样
* `Bar.prototype = new Foo()`基本满足要求，但会产生一些副作用，比如注册到其他对象，给 this 添加数据属性等
* 因此创建一个合适的关联对象使用`Object.create(..)`，它唯一的缺点就是需要创建一个新对象然后把旧对象抛弃掉，不能直接修改已有的默认对象

```JavaScript
function Foo(name) {
  this.name = name;
}
Foo.prototype.myName = function() {
  return this.name;
};
function Bar(name, label) {
  Foo.call(this, name);
  this.label = label;
}

// 我们创建了一个新的 Bar.prototype 对象并关联到 Foo.prototype
// ES6 之前需要抛弃默认的 Bar.prototype
Bar.prototype = Object.create(Foo.prototype);
// ES6 开始可以直接修改现有的 Bar.prototype
Object.setPrototypeOf(Bar.prototype, Foo.prototype);
// 注意现在没有 Bar.prototype.constructor 了

Bar.prototype.myLabel = function() {
  return this.label;
}
var a = new Bar("a", "obj a");

console.log(a.myName());  // a
console.log(a.myLabel()); // obj a
```

`检查“类”关系`
* 在 a 的整条 [[Prototype]] 链中是否有指向 Foo.prototype 的对象
* instanceof 只能处理对象和函数之间的关系

```JavaScript
a instanceof Foo;   // true
```

* 在 a 的整条 [[Prototype]] 链中是否出现出现过 Foo.prototype

```JavaScript
Foo.prototype.isPrototypeOf(a);   // true
```

* 在 ES5 中，直接获取一个对象的 [[Prototype]]链

```JavaScript
Object.getPrototyoeOf(a) === Foo.prototype;   // true
```

* 绝大多数浏览器也支持一种非标准的方法来访问内部 [[Prototype]] 属性

```JavaScript
a._proto_ === Foo.prototype;    // true
```

### 对象关联
* `Object.create(null)`会创建一个拥有空链接的对象，这个对象无法进行委托
* 这些特殊的空 [[Prototype]] 对象通常被称作`字典`，他们完全不会受到原型链的干扰，因此非常适合用来`储存数据`

`内部委托`
```JavaScript
var anotherObject = {
  cool: function() {
    console.log("cool");
  }
};
var myObject = Object.create(anotherObject);
myObject.doCool = function() {
  this.cool();  // 内部委托
};

myObject.doCool(); // cool
```
