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
