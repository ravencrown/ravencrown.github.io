---
layout: post
title: Vue 源码分析（二）：源码目录设计
date: 2018-05-09 22:39:05
tags: Vue
categories: JavaScript
---

Vue.js 的源码都在 src 目录之下，把源码 down 下来之后可以看到 src 目录下的结构

![image](https://user-images.githubusercontent.com/9835391/46247775-e265ad00-c442-11e8-9cdb-1e13864452d3.png)


## 一、compiler 目录结构

compiler 都是与编译相关的逻辑，例如 virtual-dom，template，render()等等。它包括把模板解析成 ast 语法树，ast 语法树优化，代码生成等功能。

编译的工作可以在构建时做（借助 webpack、vue-loader 等辅助插件）；也可以在运行时做，使用包含构建功能的 Vue.js。显然，编译是一项耗性能的工作，所以更推荐前者——离线编译。

![image](https://user-images.githubusercontent.com/9835391/46247795-3bcddc00-c443-11e8-907e-c438a4da0f07.png)

## 二、core 

core 目录包含了 Vue.js 的核心代码，包括内置组件、全局 API 封装，Vue 实例化、观察者、虚拟 DOM、工具函数等等。还有一些组件例如 keep-alive 等等。

这里的代码可谓是 Vue.js 的灵魂。

![image](https://user-images.githubusercontent.com/9835391/46247894-a3d0f200-c444-11e8-8e85-5c59ec212881.png)

## 三、platform

Vue.js 是一个跨平台的 MVVM 框架，它可以跑在 web 上，也可以配合 weex 跑在 native 客户端上。platform 是 Vue.js 的入口，2 个目录代表 2 个主要入口，分别打包成运行在 web 上和 weex 上的 Vue.js。

![image](https://user-images.githubusercontent.com/9835391/46247896-b2b7a480-c444-11e8-854c-fb75f02cb891.png)

## 四、server

Vue.js 2.0 支持了服务端渲染，所有服务端渲染相关的逻辑都在这个目录下。注意：这部分代码是跑在服务端的 Node.js，不要和跑在浏览器端的 Vue.js 混为一谈。

服务端渲染主要的工作是把组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将静态标记"混合"为客户端上完全交互的应用程序。

![image](https://user-images.githubusercontent.com/9835391/46247913-da0e7180-c444-11e8-8eb2-a1d5398053ba.png)

## 五、sfc

通常我们开发 Vue.js 都会借助 webpack 构建， 然后通过 .vue 单文件来编写组件。

这个目录下的代码逻辑会把 .vue 文件内容解析成一个 JavaScript 的对象。

![image](https://user-images.githubusercontent.com/9835391/46247925-f27e8c00-c444-11e8-8981-2f05da1ec49a.png)

## 六、shared

Vue.js 会定义一些工具方法或者常量，这里定义的工具方法都是会被浏览器端的 Vue.js（core目录代码、或者 platform 目录代码） 和服务端的 Vue.js（server目录） 所共享的。

![image](https://user-images.githubusercontent.com/9835391/46247928-0629f280-c445-11e8-94ce-87e3e9f05aba.png)

## 七、总结

从 Vue.js 的目录设计可以看到，作者把功能模块拆分的非常清楚，相关的逻辑放在一个独立的目录下维护，并且把复用的代码也抽成一个独立目录。

这样的目录设计让代码的阅读性和可维护性都变强，是非常值得学习和推敲的。





