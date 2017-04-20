
> # 对象

* [语法](#语法)
* [类型](#类型)
* [内容](#内容)
* [遍历](#遍历)

### 语法
* 对象可以通过两种形式定义：`声明（文字）`形式和`构造`形式
* 文字声明可以添加多个键/值对，构造形式必须逐个添加属性
* 字面形式更常用，不过有时候构造形式可以提供更多选项

```JavaScript
// 声明文字
var myObj = {
  key: value
};
// 构造函数
var myObj = new Object();
myObj.key = value;
```

### 类型
* 许多人认为`JavaScript 中万物都是对象`，这是`错误`的

`基本类型`
* string
* number
* boolean
* `null`（typeof null: oject // 这是语言本身的 bug）
* undefined

`引用类型`
* `object`（Array, Function ..）

`内置对象`
* String, Number, boolean, Object, Function, Array, Date, RegExp, Error
* 这些内置函数可以当作`构造函数`来使用
* `null`，`undefined`没有对应的构造形式，相反，`Date`只有构造
* 对于`Object`,`Array`,`Function`和`RegExp`来说，无论使用文字形式还是构造形式，它们都是对象
* `Error`对象很少在代码中显示创建，一般是在抛出异常时被自动创建

```JavaScript
var strPrimitive = "I am a string";
console.log(typeof strPrimitive);  // string
console.log(strPrimitive instanceof String);  // false

var strObject = new String("I am a string");
console.log(typeof strObject);  // Object
console.log(strObject instanceof String);   // true

console.log(Object.prototype.toString.call(strObject));  // [object String]
```

* 引擎自动把字面量转化成 String 对象

```JavaScript
var strPrimitive ="I am a string";
console.log(strPrimitive.length);     // 13
console.log(strPrimitive.charAt(3));  // m
```

### 内容
访问 myObject 中 a 位置上的值
* .操作符：`属性访问`，要求属性名满足标识符的命名规范，如果引用名称为`Super-Fun`，则使用键访问
* []操作符：`键访问`

在对象中，`属性名`永远都是`字符串`
```JavaScript
var myObject = {};
myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

console.log(myObject["true"]);  // foo
console.log(myObject["3"]);     // bar
console.log(myObject["[object Object]"]);  // baz
```

`可计算属性名`
```JavaScript
var prefix = "foo";
var myObject = {
  [prefix + "bar"]: "hello",
  [prefix + "baz"]: "world"
};

console.log(myObject["foobar"]); // bar
console.log(myObject["foobaz"]); // baz
```

`属性和方法`
* 即使你在对象的文字形式中声明一个函数表达式，这个函数也不会”属于“这个对象 -- `它们只是对于相同函数对象的多个引用`

```JavaScript
var myObject = {
  foo: function() {
    console.log("foo");
  }
};
var somefoo = myObject.foo;

console.log(somefoo);       // [Function foo()]
console.log(myObject.foo);  // [Function foo()]
```

`数组`
* 数组也是对象，虽然每个下标都是整数，你仍然可以`给数组添加属性`
* 最好只用对象来存储键/值对，只用数组来存储数值下标/值对
* 如果你试图向数组添加一个属性，但是属性名”看起来“像一个数字，那它会变成一个数值下标

```JavaScript
var myArray = ["foo", 42, "bar"];

myArray.baz = "baz";
console.log(myArray.length); // 3
console.log(myArray.baz);    // baz

myArray["3"] = "baz";
console.log(myArray.length); // 4
console.log(myArray[3]);     // baz
```

`复制对象`
* 相对`JSON`安全的对象来说，有一种巧妙的复制方法

```JavaScript
var newObj = JSON.parse(JSON.stringify((someObj)));
```

* ES6 定义了`Object.assign(..)`方法来实现`浅复制`
* Object.assign(..) 就是使用 = 操作符来赋值，所以`源对象`属性的一些`特性`(比如 writable)不会被复制到`目标对象`
* 对于`浅复制`，复制出的新对象中的 a 的值会复制旧对象中 a 值，但是新对象中的 b, c, d 三个属性其实只是三个引用
* 而`深复制`除了复制 myObject 以外，还会复制 anotherObject 和 anotherArray，这时问题就来了，anotherArray 引用了 anotherObject 和 myObject，所以又需要复制 myObject，这样就会由于循环引用导致死循环

```JavaScript
function anotherFunction() {}
var anotherObject= {
  c: true
};
var anotherArray = [];
var myObject = {
  a: 2,
  b: anotherObject,
  c: anotherArray,
  d: anotherFunction
};
anotherArray.push(anotherObject, myObject);

var newObj = Object.assign({}, myObject);
newObj.a;   // 2
console.log(newObj.b === anotherObject);    // true
console.log(newObj.c === anotherArray);     // true
console.log(newObj.d === anotherFunction);  // true
```

`属性描述符`
* 一个普通的对象属性对应的属性描述符还包含了另外三个特性：`writable`（可写），`enumerable`（可枚举），`configurable`（可配置）

```JavaScript
var myObject = {
  a: 2
};

console.log(Object.getOwnPropertyDescriptor(myObject, "a"));
// { value: 2, writable: true, enumerable: true, configurable: true }
```

* 我们可以使用`Object.defineProperty(..)`来添加一个新属性或者修改一个已有属性并对特性进行设置
* `writable`：是否可以修改属性的值
* `configurable`：是否可以修改属性描述符，且把 configurable 修改成 false 是`单向操作`，无法撤销，但即便属性是 configurable: false，我们还是可以把`writable`的状态由 true 改为 false，但是无法由 false 改为 true，除了无法修改，configurable: false 还会禁止删除这个属性
* `enumerable`：属性是否可枚举

```JavaScript
var myObject = {
  a: 2
};

console.log(myObject.a);  // 2
delete myObject.a;
console.log(myObject.a);  // undefined

Object.defineProperty(myObject, "a", {
  value: 2,
  writable: true,
  configurable: false,
  enumerable: true
});

console.log(myObject.a);  // 2
delete myObject.a;
console.log(myObject.a);  // 2
```

`不变性`
* 所有的方法创建的都是`浅不变形`，也就是说，它们只会影响目标对象和它的直接属性

```JavaScript
myImmutableObject.foo;  // [1, 2, 3]
myImmutableObject.foo.push(4);
myImmutableObject.foo;  // [1, 2, 3, 4]
```

* `对象常量`

```JavaScript
var myObject = {};

Object.defineProperty(myObject, "FAVORITE_NUMBER", {
  value: 42,
  writable: false,
  configurable: false
});
```

* `禁止扩展`

```JavaScript
var myObject = {
  a: 2
};
Object.preventExtensions(myObject);

myObject.b = 3;
console.log(myObject.b);  // undefined
```

* `密封`

```
Object.seal(..) 会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用 Object.preventExtensions(..) 并把所有现有属性标记为 configurable: false
```

* `冻结`

```
Object.freeze(..) 会创建一个冻结的对象，这个方法实际上会在一个现有对象上调用 Object.seal(..) 并把所有“数据访问”属性标记为 writable: false
```

`[[Get]]`
* 在语言规范中，myObject.a 在 myObject 上实际上是实现了 [[Get]] 操作
* 对象默认的内置 [[Get]] 操作首先在对象中查找是否有名称相同的属性，如果找到就会返回这个属性的值
* 然而，如果没有找到名称相同的属性，则会遍历`原型链`
* 如果还是没找到，则返回 undefined

`[[Put]]`
* 属性是否是`访问描述符`，如果是并且存在`setter`就调用 setter
* 数据的数据描述符中`writable`是否为`false`，如果是在非严格模式下`静默失败`，在严格模式下抛出`typeError`异常
* 如果都不是，将该值设置为属性的值

`Getter 和 Setter`
* 当你给一个属性定义 getter 和 setter 时，这个属性会被定义为`访问描述符`，JavaScript 会忽略它们的`value`和`writable`属性
* 不管是对象文字语法中的 get a() {..}，还是 defineProperty(..) 中显式定义，二者都会在对象中创建一个不包含值的属性，对于这个属性的访问会自动调用一个`隐藏函数`，它的返回值会被当作`属性访问`的返回值

```JavaScript
var myObject = {
  set a(val) {
    this._a_ = val;
  }
};
myObject.a = 2;
Object.defineProperty(
  myObject,
  "b",
  {
    // 描述符
    get: function() {
      return this._a_ * 2;
    },
    // 确保 b 会出现在对象的属性的列表中
    enumerable: true
  }
);

console.log(myObject.b);  // 4
```

`存在性`
* `propertyIsEnumerable(..)`会检查给定的属性名是否直接存在于对象中，并且满足 enumerable: true
* `Object.keys(..)`会返回一个数组，包含所有可枚举的属性
* `Object.getOwnPropertyNames(..)`会返回一个数组，包含所有属性，无论它们是否可枚举
* `in`和`hasOwnProperty(..)`的区别在于是否查找原型链，然而，`Object.keys(..)`和`Object.getOwnPropertyNames`都只会查找对象直接包含的属性
* 目前并没有内置的方法可以获取`in`操作符使用的属性列表（对象本身的属性以及原型链中的所有属性）
* `in`操作符可以检查容器内是否有某个值，但实际上检查的是某个属性名是否存在

```JavaScript
var myObject = {};
Object.defineProperty(
  myObject,
  "a",
  {
    value: 2,
    enumerable: true
  }
);
Object.defineProperty(
  myObject,
  "b",
  {
    value: 3,
    enumerable: false
  }
);

console.log(myObject.propertyIsEnumerable("a"));  // true
console.log(myObject.propertyIsEnumerable("b"));  // false

console.log(Object.keys(myObject));   // ["a"]
console.log(Object.getOwnPropertyNames(myObject));  // ["a", "b"]

console.log("a" in myObject); // true
console.log(myObject.hasOwnProperty("a"));  // true

console.log(4 in [2, 4, 6]);  // false
console.log(1 in [2, 4, 6]);  // true
```

### 遍历
ES6 的`for..of`语法可以遍历数据结构中（数组，对象等）的值，for..of 会寻找内置或者自定的的 @@iterator 对象并调用它的 next() 方法来遍历数据值
```JavaScript
var myArray = [1, 2, 3];
var it = myArray[Symbol.iterator]();

console.log(it.next());  // { value: 1, done: false }
console.log(it.next());  // { value: 1, done: false }
console.log(it.next());  // { value: 1, done: false }
console.log(it.next());  // { value: undefined, done: true }
```
