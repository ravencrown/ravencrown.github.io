---
layout: post
title: Vue 源码分析（01）之 vue-flow
date: 2018-05-09 20:31:12
tags: Vue
categories: JavaScript
---

Flow 是 facebook 出品的 JavaScript 静态类型检查工具，Vue 的源码采用了 Flow 做静态类型检查，所以要熟悉 Vue 的源码分析，首先要了解 Flow。

Flow 有两种类型检查

> - 类型推断：通过变量的使用上下文来推断出变量类型，然后根据这些推断来检查类型。
> - 事先注释好我们期待的类型，Flow 会基于这些注释来判断。

看以下例子

```js
/*@flow*/``

// function split(str) {
//     return str.split(' ')
// }
  
// split(11) // Cannot call str.split because property split is missing in Number [1]

/** -------- */

/*@flow*/
// function add(x, y){
//     return x + y
// }

// add('Hello', 11) // no error


/*@flow*/
// function add(x: number, y: number): number {
//     return x + y
// }

// add('Hello', 11) // Cannot call add with 'Hello' bound to x because string [1] is incompatible with number [2].

/*@flow*/

// var arr: Array<number> = [1, 2, 3]

// arr.push('Hello') //Cannot call arr.push because string [1] is incompatible with number [2] in array element.


/*@flow*/

class Bar {
    x: string;           // x 是字符串
    y: string | number | void;  // y 可以是字符串或者数字
    z: boolean;

    constructor(x: string, y: string | number | void) {
        this.x = x
        this.y = y
        this.z = false
    }
}

var bar: Bar = new Bar('hello', 4)

var obj: { a: string, b: number, c: Array<string>, d: Bar } = {
    a: 'hello',
    b: 11,
    c: ['hello', 'world'],
    d: new Bar('hello', 3)
}

```

## 参考资料

[官网](https://flow.org/en/docs/install/)
[Type Annotations](https://flow.org/en/docs/types/)


