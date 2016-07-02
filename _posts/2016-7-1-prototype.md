---
layout: post
title:  "Class vs Prototype 光与暗 上卷"
date:   2016-7-1 8:00:00
categories: javascript
---

书到了，接下来搞清楚自己一直挺迷糊的原型链（prototype chain）。

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain) 说：

>在 javaScript 中，每个对象都有一个指向它的原型（prototype）对象的内部链接。这个原型对象又有自己的原型，直到某个对象的原型为 null 为止（也就是不再有原型指向），组成这条链的最后一环。这种一级一级的链结构就称为原型链（prototype chain）。

本身并没有太难理解的概念，但原型链的问题在面试里会跟着类的实现被提出来。

没错类是需要自己实现或者说模拟的（ES6 的 class 关键字可以视作一种官方实现），JavaScript 里全是对象，根本没有内置的类。

为什么大家都对类这么执着呢？我想象到的是 Java 或 C++ 程序员们在面对残缺的 js 项目时，苦心地把原来的工程思维模式带过来的画面，他们可能不会想到，几年前当一个 js 入门的编程初学者看到一本本书拿出各种其他语言的主流范式来给 js 作类比有多困惑，就像一本最新重启的漫威漫画通常都默认你早对主宇宙对应的人物起源故事都了然于心了。

好吧，我了解对于老读者来说，看本叔再死一次会觉得浪费纸张这种感觉。作为从 js 入门的初学者就应该正视这些问题，还有随之而来的通常是基于类来实现的种种高级设计模式和最佳实践。

然而 “类这种设计模式不适合 js”，这本《你不知道的 JavaScript 上卷》后三分之一大概都在说这件事……

<br>

现在，问题已经比 “为什么变量提升” 复杂得多了！

最开始我要改变的观念是，原型链并不生来就为了实现继承和类模型的其他功能，<q>这个机制的本质是对象之间的关联关系</q>。

> 如果一个对象上没有找到需要的属性或者方法引用，引擎就会继续在 [[Prototype]] 关联的对象上进行查找。

```javascript
const obj1 = {
  a: 2,
};

const obj2 = Object.create(obj1);
console.log(obj2.a); // 2
```

这个例子里用了 [`Object.create`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create) 来直接指定 prototype。在 ES5 之前，这件事需要这样做

```javascript
var obj1 = {
  a: 2
};

var obj2 = (function() {
  function F() {};
  F.prototype = obj1;
  return new F();
})();

console.log(obj2.a); // 2
```

顺便提，ES6 里还有个更直接的方法（据说有性能问题）

```javascript
const obj1 = {
  a: 2,
};

const obj2 = Object.setPrototypeOf({}, obj1);

console.log(obj2.a); // 2
```

其实第二种方法做了多余的事情，因为 new 操作符的实际作用是 “将一个函数作为构造函数调用”, 一个空的构造函数被执行了，这又如何？prototype 上还有个属性叫 constructor

```javascript

const obj1 = {
  a: 2,
};

const obj2 = Object.create(obj1);

const obj3 = (function() {
  function F() {};
  F.prototype = obj1;
  return new F();
})();

console.log(obj2.constructor); // function Object {}

console.log(obj3.constructor); // function F() {}
```

额，这个 constructor 看起来是指向实例的构造函数的引用。MDN 上中文版似乎有个漏洞，constructor 只在原始类型上只读。也就是说其他情况下这个值是可写的。

```javascript
Date.prototype.constructor = Number;

a = new Date();

console.log(a.constructor); // Number
console.log(a instanceof Date); // true
```

看起来不怎么可靠呢。还是按这本书所说

> 最好记住这一点 “constructor 并不表示被构造”。

> ……实际上是不被信任的，他们不一定会指向默认的函数引用。很快我们就会看到，稍不留神 constructor 就可能会只想你意想不到的地方。

<br>

那好，我们来看看到底可能发生什么。我们现在知道原型链的机制了，可以用 new 或者 Object.create 来创建它，现在我们用这种机制来模拟类模型里的继承。为什么是模拟而不是实现呢？因为类继承是一种复制操作，而 js 原生并不提供深度复制方法。


```javascript
function Foo(name) {
  this.name = name;
}

Foo.prototype.getName = function() {
  return this.name;
};

function Bar(name, gender) {
  Foo.call(this, name);
  this.gender = gender;
}

Bar.prototype = new Foo();

Bar.prototype.getGender = function() {
  return this.gender;
};

const bar = new Bar('gogu', 'male');

bar.getName(); // 'gogu'
bar.getGender(); // 'male'
```

哦！还记得不被信任的 constructor 么…… `Bar.prototype = new Foo()` 导致 `Bar.prototype.constructor === Foo.prototype.constructor === Foo` 进而

```javascript
bar instanceof Bar; // true
bar.constructor; // Foo;

// 只好手动修复，虽然觉得没什么意义
bar.constructor = Bar;
```

一般原型继承风格的代码这样就可以了，不过还有个瑕疵。注意到为了创建 Foo 与 Bar 之间的原型链，而在描述 Bar 类的过程中就用 new 关键字实例化了一次 Foo 类了吗，这意味着 Foo 的构造函数在我们没有真的想创建 Foo 或 Bar 的实例化对象时就被执行了，这次和之前单纯创建原型链的例子不同，执行的可不是一个空的构造函数了！视代码的腐败程度这会带来各种计划之外的副作用。

所以有 Object.create 的情况下（ES5 或 polyfill），建议不要用 new 来创建原型链。

```javascript
function Foo(name) {
  this.name = name;
}

Foo.prototype.getName = function() {
  return this.name;
};

function Bar(name, gender) {
  Foo.call(this, name);
  this.gender = gender;
}

Bar.prototype = Object.create(Foo.prototype); // 直接关联 prototype 的方式

Bar.prototype.constructor = Bar; // 还是得手动修复 ……

Bar.prototype.getGender = function() {
  return this.gender;
};

const bar = new Bar('gogu', 'male');

bar.getName(); // 'gogu'
bar.getGender(); // 'male'
```

ES6 之后我们可以用 class 和 extends 关键词来避免这一系列琐碎的过程和陷阱了

```javascript
class Foo {
  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

class Bar extends Foo {
  constructor(name, gender) {
    super(name);
    this.gender = gender;
  }

  getGender() {
    return this.gender;
  }
}


const bar = new Bar('gogu', 'male');

bar.getName(); // 'gogu'
bar.getGender(); // 'male'
bar.constructor; // Bar


Foo.prototype.isPrototypeOf(Bar.prototype); // true 所以 class 和 extends 就是基于原型链继承的官方实现语法糖
```

既然 ES6 已经让模拟类可以做得如此清爽，那么《你不知道的 JavaScript 上卷》为什么依然在说

> ……在 JavaScript 中模拟类是得不偿失的，虽然能解决当前的问题，但是可能会埋下更多的隐患。

js 模拟类的本质就是用联系来模拟复制的表象。但其实有联系就已经够强大了，不需要让事情变得更复杂。在书中，作者鼓励一种丢掉类模型的思考模式，叫做面向委托的设计。如上功能，实现起来应该是这样

```javascript
const Foo = {
  init(name) {
    this.name = name;
  },
  getName() {
    return this.name;
  },
};

const Bar = Object.create(Foo);

Bar.setup = function(name, gender) {
  this.init(name);
  this.gender = gender;
};

Bar.getGender = function() {
  return this.gender;
};

const bar = Object.create(Bar);
bar.setup('gogu', 'male');

bar.getName(); // 'gogu'
bar.getGender(); // 'male'
```

确实没有了遮遮掩掩，个人初感笨拙却做回了自己，值得去实践一下。

事实上书的附录中还痛批了 ES6 的 class 关键字，说其隐藏了本质，增加了误解。见仁见智吧，我觉得无论用 js 或者无论什么语言实现了什么样的范式都不应该算犯错，也许就是开发者口味和市场供求关系的问题。