# this 和对象原型

## 前言

平时在工作中用原型和`this`极少，但在看别人库时因为不熟悉这块内容，看得比较痛苦，所以还是要系统的学习下`javascirpt`原生的内容。

### 概述

`javascript`原生只有对象，没有类这个概念，`new`运算符，es6的`class`只是对面向对象模式的模拟，`javascript`内部是通过原型对象和原型链的机制来实现对象属性和方法的复用。所以在我们设计程序时，**"OLOO"**（objects-linked-to-other-objects）是比面向对象（class-orientation）更自然的方式。

### 函数与对象

`javascrpt`原始类型中没有函数类型，函数是属于复杂原始类型对象的子类型。我们用instanceof来判断。

>instanceof 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上。

```javascript
const Foo = function(){}

const foo = new Foo // {}

foo instanceof Foo // true
Foo instanceof Object // true

Foo.prototype // {constructor: Foo()}
Foo.prototype === Object.getPrototypeOf(foo) // true
```

可以看到`foo`的原型链上游存在构造函数(本质就是普通函数)`Foo`的原型属性。`Foo`函数有原型属性prototype，还有`call`、`apply`等属性方法，表明函数是对象的子类型。

那为什么`Foo instanceof Object`也为`true`呢？我们可以用对象的`__proto__`属性（等价于`Object.getPrototypeOf`方法）来探索，这两种方式都可以获取到对象原型链的上一级对象原型对象。

```javascript
Foo.__proto__ // ƒ () { [native code] }
Foo.__proto__ === Function.prototype // true 函数的原型对象
Foo.__proto__.call // ƒ call() { [native code] } call等方法都在这里

Foo.prototype.__proto__ // {constructor: ƒ, __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, __lookupGetter__: ƒ, …} 对象的原型对象
```
