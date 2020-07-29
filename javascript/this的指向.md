# this的指向

## 概述

`this`是一个对象，一般在函数中使用。我们常常感到疑惑`this`是指的哪个对象。在`javascript`中，一般`this`的指向都由调用函数的环境决定的，所以我们要认识到`this`的不确定性。

## 确定`this`指向的4条规则

### 1. 默认绑定

在函数直接调用时，`this`默认指向全局对象。

```javascript
  let foo = function() {
    console.log(this)
  }

  foo() // Window {...}
```

### 2. 隐式绑定

调用函数的上下文有对象，那么该函数隐式地绑定在它调用时，上下文中距它最近的对象上。

```javascript
  function foo() {
    console.log( this.a );
  }

  let obj2 = {
    a: 33,
    foo: foo,
  }

  let obj = {
    a: 2,
    obj2: obj2,
  };

  let a = 'global'
  let bar = obj2.foo

  obj2.foo(); // 33
  obj.obj2.foo() // 33
  bar() // global
  setTimeout( obj2.foo, 100 ); // global
```

`obj.obj2.foo()`打印出`33`，说明如果函数在对象深层属性里调用，是绑定在它最近的对象上。

而`bar()`打印出`global`看起来绑定丢失了，但其实函数是引用赋值，这里跟默认绑定的情况一样，`this`指向的是`Window`，离它调用时最近的对象也是`Window`。

最需要注意的是`setTimeout( obj.foo, 100 );`这种接受一个函数作为参数的情况。该参数在调用时，相当于在`setTimeout`函数体里调用`foo`，上下文对象是全局。

### 3. 显式绑定

函数内置方法`call`，`apply`，`bind`可以为函数绑定`this`。

```javascript
  function foo(num) {
    console.log( this.a + num );
  }

  let obj = {
    a: 2
  };

  foo.call( obj, 3 ); // 5
  foo.apply( obj, [3] ) // 5

  let fooBind = foo.bind(obj)
  fooBind(3) // 5

  let obj2 = {
    a: 3
  }
  fooBind.bind(obj2)(3) // 5
```

可以看出多次`bind`，`this`指向第一次绑的对象。

### 4. `new`操作符绑定

我们先看`new`操作符的实现流程：

1. 新建一个新对象。
2. 将该对象的原型对象设置为构造函数的原型。
3. 构造函数显示绑定新对象，然后调用。
4. 如果构造函数返回值为对象，则返回构造函数返回值，否则返回该新对象。

所以很明显，构造函数里的`this`绑定在返回的对象上。

### 4条规则优先级

`new`操作符 > 显式绑定 > 隐式绑定 > 默认绑定

## 箭头函数this的指向

普通函数中`this`的指向是运行时确定的，所以在定义时无法确实指向。但箭头函数不一样，它在定义时就确定了`this`的指向，不会随运行时上下文的改变而改变。

> 箭头函数不会创建自己的this,它只会从自己的作用域链的上一层继承this

因为`javascript`是函数作用域，所以箭头函数的`this`为上层函数的`this`。

```javascript
  let obj = {
    a: 1,
    foo: () => {
      console.log(this)
    }
  }
  obj.foo() // Window {...}, 所以foo的作用链上层是全局

  let Foo = function() {
    this.a = 4
    let f = () => {
      console.log(this)
    };
    f()
  }
  Foo() // Window {...} 直接调用Foo，this默认绑定全局
  new Foo() // Foo {a: 4} new操作符，Foo里的this绑定创建的对象，箭头函数f里的this是上层作用域Foo的this，即new创建的对象。
```

箭头函数比普通函数更加纯洁，专注于函数的本质：输入 ->输出。它没有如下几个普通函数特性：

* 它没有`this`的负担，会忽略`call`、`apply`、`bind`等绑定`this`的方法的第一个参数，所以无法绑定`this`，所以像这种看起来很复杂的代码：

```javascript
  function foo() {
    return () => {
      return () => {
        return () => {
          console.log("id:", this.id);
        };
      };
    };
  }

  var f = foo.call({id: 1});

  var t1 = f.call({id: 2})()(); // 1
  var t2 = f().call({id: 3})(); // 1
  var t3 = f()().call({id: 4}); // 1
```

只要用[babel](https://babeljs.io/repl)翻译翻译

```javascript
  function foo() {
    var _this = this;

    return function () {
      return function () {
        return function () {
          console.log("id:", _this.id);
        };
      };
    };
  }

  var f = foo.call({id: 1});

  var t1 = f.call({id: 2})()();
  var t2 = f().call({id: 3})();
  var t3 = f()().call({id: 4});
```

就很容易看出，`console.log`里的`this`是`foo`的`this`，因为只有foo是普通函数，`foo.call()`才能将对象`{id: 1}`绑定到`foo`的`this`上，中间返回的都是箭头函数，无论再怎么调用`call`，都没法将对象绑定到`this`上。

* 箭头函数不能用作构造器，和 `new` 一起用会抛出错误。

* 箭头函数没有prototype属性。

* yield 关键字通常不能在箭头函数中使用（除非是嵌套在允许使用的函数内）。因此，箭头函数不能用作函数生成器。

## 参考

* [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

* [You Dont Know JS](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch2.md)

* [ES6 入门教程 issue](https://github.com/ruanyf/es6tutorial/issues/150)