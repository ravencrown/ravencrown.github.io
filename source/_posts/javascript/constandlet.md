---
layout: post
title: 全局作用域中，用const和let声明的变量去哪了？
date: 2019-03-03 11:31:20
tags: JavaScript
categories: JavaScript
---

let、const声明的变量，暴露在全局，为什么没挂载到window下？究竟挂载到哪里去了？

我们打开控制台，输入

```js
const a = 123;
function abcd() {
    console.log(a);  // abc函数的作用域能访问到a
};
dir(abcd);
```
可以在 [[Scopes]] 属性中

![image](https://user-images.githubusercontent.com/9835391/53818757-8e3b9780-3fa3-11e9-8749-8b3b633092a2.png)


const、let 这类都是属于“Declarative Environment Records” 声明性环境记录，和函数、类这些一样，在单独的存储空间，var这类，属于“object environment record”，会挂载到某个对象上，也会沿着原型链去向上查找

说明不挂载到对象上，但是在一个上下文的方法中，可以访问到let、const 声明记录






