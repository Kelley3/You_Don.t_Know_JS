
> # 行为委托

* [面向委托的设计](#面向委托的设计)

### 面向委托的设计
`类理论`
* `类设计模式`鼓励你在继承时使用方法`重写`（和多态）

```JavaScript
class  Task {
  id;
  Task(ID) { id = ID; }
  outputTask() { output(id); }
}

class  XYZ  inherits Task {
  label;
  XYZ(ID, Label) { super(ID); label = Label; }
  outputTask() { super(); output(label); }
}
```

`委托理论`
* 在 JavaScript 中，`[[Prototype]]`机制会把对象关联到其他对象

```JavaScript
Task = {
  setID: function(ID) { this.id = ID; },
  outputID: function() { console.log(this.id); }
};

// 让 XYZ 委托 TASK
XYZ = Object.create(Task);

XYZ.prepareTask = function(ID, Label) {
  this.setID(ID);
  this.label = label;
};

XYZ.outputTaskDetails = function() {
  this.outputID();
  console.log(this.label);
};
```
* 在 [[Prototype]] 委托中最好把状态保存在`委托者`（XYZ）而不是`委托目标`（Task）上
* 在委托行为中，尽量`避免`[[Prototype]] 链的不同级别中`使用相同命名`
* 和 XYZ 交互时，可以使用 Task 中的通用方法

`原型面向对象风格`
```JavaScript
function Foo(who) {
  this.me = who;
}
Foo.prototype.identify = function() {
  return "I am " + this.me;
};

function Bar(who) {
  Foo.call(this, who);
}
Bar.prototype = Object.create(Foo.prototype);
Bar.prototype.speak = function() {
  console.log("Hello, " + this.identify() + ".");
};

var b1 = new Bar("b1");
var b2 = new Bar("b2");

b1.speak(); // Hello, I am b1.
b2.speak(); // Hello, I am b2.
```

`对象关联风格`
```JavaScript
Foo = {
  init: function(who) {
    this.me = who;
  },
  identify: function() {
    return "I am " + this.me;
  }
};

Bar = Object.create(Foo);
Bar.speak = function() {
  console.log("Hello, " + this.identify() + ".");
};

var b1 = Object.create(Bar);
b1.init("b1");
var b2 = Object.create(Bar);
b2.init("b2");

b1.speak(); // Hello, I am b1.
b2.speak(); // Hello, I am b2.
```
