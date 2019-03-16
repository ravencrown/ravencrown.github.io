---
layout: post
title: JavaScript 编译原理
date: 2019-02-16 11:31:20
tags: JavaScript
categories: JavaScript
---

程序执行一段源代码之前会经历三个步骤

分词/词法分析
例如 `var a = 2`，会被分解成 `var、a、=、2`

解析/语法分析
这个过程是将词法单元流（数组）转换成由一个元素逐级嵌套所组成的代表了程序语法结构的树，称为抽象语法树`（Abstract Syntax Tree， AST）`

例如 `var a = 2` 的抽象语法树有一个叫做 `VariableDeclaration` 的顶级节点，接下来是 Identifier 的子节点（它的值是a），以及一个叫 `AssignmentExpression` 的子节点。`AssignmentExpression` 节点有一个叫做 `NumericLiteral` 的子节点（它的值是2）

代码生成
将 AST 转换为可执行代码的过程被称为代码生成。

抛开具体细节，简单的来说就是有某种方法可以将 `var a = 2` 的AST转化为一组机器指令，用来创建一个叫做 a 的变量（包括分配内存），并将一个值存储在 a 中


