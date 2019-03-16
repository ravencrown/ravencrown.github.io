---
layout: post
title: 查询作用域导致的异常
date: 2019-03-02 11:31:20
tags: JavaScript
categories: JavaScript
---


不成功的 `RHS` 引用会导致抛出 `ReferenceError`异常，不成功的`LHS`引用会导致自动隐式地创建一个全局变量（非严格模式下），该变量使用`LHS`引用的目标作为标识符，或者抛出`ReferenceError`（严格模式下）

如果 `RHS` 查询找到了一个变量，但是尝试对这个变量进行不合理的操作，比如试图对一个非函数类型的值进行函数调用，或者引用 `null` or `undefined` 类型的值中的属性，那么引擎会抛出另外一种类型的异常，叫做`TypeError`

`ReferenceError` 同作用域判别失败相关
`TypeError` 代表作用域判别成功了，但是对结果的操作是非法或者不合理的

**tips**
严格模式下禁止自动或隐式地创建全局变量








