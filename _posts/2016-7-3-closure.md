---
layout: post
title:  "在 closure 的阴影里"
date:   2016-7-3 8:00:00
categories: javascript
---

ES6 的箭头函数我是非常喜欢用，尤其了解了一些函数式编程思维后，觉得写的代码很魔性。于是上次的面试中问道箭头函数时我内心是欢跃的。结果 this 指向的问题我解释成了箭头函数会自动 `bind(this)`，对面深沉了一下，问道如果箭头函数被 call 调用呢？我说那可以不用箭头函数啊，不过肯定是按 call 来的没错！对面只是邪魅地笑笑。却！没！告！诉！我！有！问！题！回家后我在 babel 官网上试了一下它到底怎么处理箭头函数的…… NONONO 虽然他没义务告诉我但是我不想和这个人共事！至少到这件事的阴影过去……

```javascript
// ES6
const b = function() {
  const a = () => this;  
}

// after Babel
var b = function b() {
  var _this = this;

  var a = function a() {
    return _this;
  };
};
```

用闭包（closure）特性获得当前作用域下 this 的引用，并不是我想的 bind(this) 。实际上 ES6 的规格里，箭头函数就是词法性地绑定当前作用域下的 this 值（还有 arguments，super 等），是没有自己作用域下的 this 的。所以不存在会改变它的情况，变也是词法作用域下的 this 引用在变。

说说闭包的坑。我觉得我闭包原来掌握的也还不错，现在更系统地复习了一下。

首先要说词法（Lexical）作用域。js 的作用域不是动态的，而是在编译期的词法分析阶段根据函数声明时所处的位置决定。

```javascript
var a = 1;

function foo() {
  console.log(a);
}

function bar() {
  var a = 2;
  foo();
}

bar(); // 1
```

根据声明的位置输出了全局变量中的 a，而不是根据调用的时机输出 bar 作用域下的 a。

函数的变量可以在当前函数范围内使用，可是通过词法作用域的规则，我们可以得到另一作用域的引用。

```javascript
function foo() {
  const a = 2;

  return () => {
    console.log(a);
  }
}

const baz = foo();
baz(); // 2

// 另一个例子
function foo(fn) {
  const a = 3;
  fn();
}

function bar() {
  const a = 2;

  function baz() {
    console.log(a);
  }

  foo(baz); // baz 作为参数出入 foo 在调用，并不妨碍 baz 在其声明位置获得 a
}

bar(); // 2
```

道理在于一个函数被声明在另一个词法作用域中，意味着定义这个作用域总是可以被其使用，需要一直存在内存中。这种跨作用域的引用叫闭包。

闭包的玩法有很多，比如可以实现模块和单例模式（Singleton）

```javascript
var UserStore = (function() {
  var _data = [];

  return {
    add(item) {
      _data.push(item);
    },
    get(id) {
      return _data.find(d => d.id === id);
    },
  };
}());
```

一个经典的闭包坑如下

```javascript
for (var i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i);
  }, i * 1000);
}  // 每个 5 秒 log 一个 6
```

这是因为当第一次间隔一秒后, timer 函数在声明位置的词法作用域下找到的 i 引用时，循环就已经结束了。我们想要的其实是这样：

```javascript
for (var i = 1; i <= 5; i++) {
  (function() {
    var j = i;

    setTimeout(function timer() {
      console.log(j);
    }, j * 1000);
  })();
}
```

每次循环都会生成一个新的匿名函数作用域，这样每次 timer 获取的 j 是不同的引用。ES6 不出意外给出了更清爽的解决方案

```javascript
for (let i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i);
  }, i * 1000);
}
```

循环定义中使用 let，会创建隐含的块作用域，同样是会得到不同引用。 