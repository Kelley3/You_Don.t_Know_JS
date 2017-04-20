
> # this 词法

cool() 函数`丢失`了同 this 之间的绑定
```JavaScript
var obj = {
  id: "awesome",
  cool: function collFn() {
    console.log(this.id);
  }
};

var id = "not awesome";
obj.cool(); // awesome
setTimeout(obj.cool, 100);  // not awesome
```

使用`词法作用域：var self = this`
```JavaScript
var obj = {
  count: 0,
  cool: function collFn() {
    var self = this

    if (self.count < 1) {
      setTimeout(function timer() {
        self.count++;
        console.log("awesome?");
      }, 100);
    }
  }
};

obj.cool(); // awesome?
```

使用`箭头函数`，箭头函数用`当前的词法作用域`覆盖了this本来的值
```JavaScript
var obj = {
  count: 0,
  cool: function collFn() {
    if (this.count < 1) {
      setTimeout(() => {
        this.count++;
        console.log("awesome?");
      }, 100);
    }
  }
};

obj.cool(); // awesome?
```

使用`bind()`
```JavaScript
var obj = {
  count: 0,
  cool: function collFn() {
    if (this.count < 1) {
      setTimeout(function timer() {
        this.count++;
        console.log("awesome?");
      }.bind(this), 100);
    }
  }
};

obj.cool(); // awesome?
```
