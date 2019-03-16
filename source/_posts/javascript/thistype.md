---
layout: post
title: this 的四种绑定方式
date: 2019-03-05 11:31:20
tags: JavaScript
categories: JavaScript
---

- 默认绑定
- 显示绑定
- 隐式绑定
- new 绑定

其他非正式的绑定

- 硬绑定/软绑定
- 箭头绑定


## 默认绑定

函数不带任何修饰的函数引用（上下文）进行调用的。
```js
function foo {
    console.log(this.a)
}
var a = 2;
foo() // 2
```

但是如果使用严格模式，则不能将全局对象用于默认绑定，因此 `this` 会绑定到 `undefined`

```js
function foo {
    "use strict"
    console.log(this.a)
}
var a = 2;
foo() //  TypeError: this is undefined
```

兼容
```js
function foo {
    console.log(this.a)
}
var a = 2;

(function {
    "use strict";
    foo()  // 2
})()
```

## 隐式绑定

这一条考虑的是调用位置是否有上线对象

```js
function foo(){
    console.log(this.a)
}

var obj2 = {
    a: 10,
    foo: foo
}

var obj1 = {
    a: 20,
    foo: foo
}

obj1.obj2.foo(); // 10
```

如果这么思考的话，可以认为默认绑定的上下文是window
```js
function foo {
    console.log(this.a)
}
var a = 2;
foo() // 上下文是 window，这里可以看做  window.foo()
```

在全局作用域中声明 `foo`，实际上等于 声明`window.foo`


### 隐式丢失

```js
function foo(){
    console.log(this.a)
}

var obj = {
    a: 10,
    foo: foo
}

var bar = obj.foo;
var a = "oops, global";
bar(); // "oops, global"
```

此时的`bar()`，实际上是不带任何修饰函数调用，因此使用默认绑定

## 显示绑定

call, apply 等绑定


##  this 绑定

首先重新定义 `JavaScript`中的构造函数。

在JavaScript中，构造函数指的是一些使用 `new` 操作符时被调用的函数。他们并不属于某个类，也不会实例化一个类。实际上，他们甚至不能说是一种特殊的函数类型，他们只是被 `new` 操作符调用的普通函数

使用`new`来调用函数，或者说发生构造函数调用时候，会自动执行下面的操作

- 创建（或者说构造）一个全新的对象 
- 这个新对象会被执行[[ Prototype ]] 连接
- 这个新对象会被绑定到函数调用的this
- 如果函数没有返回其他对象，那么 new 表达式的函数调用会自动返回这个新对象


## 优先级

1. new 绑定
2. 显示绑定
3. 隐式绑定
4. 默认绑定


