---
layout: post
title: Node 学习笔记02 - node 编程基础
date: 2018-10-27 20:54:43
tags: Node
categories: Node
---

Node 的开发环境非常容易设置，比 Java、Python 等语言更容易配置些。如果用 Mac, 可以直接使用 HomeBrew 安装。

```bash
$ brew install node
```

## 一、NVM

个人建议使用 nvm 安装 Node。 Node 版本很多，你一定会有切换 Node 的版本的场景。nvm 就是用来管理 Node 版本的工具。另一个工具是 n 工具，不过个人喜欢使用 nvm 进行管理。[nvm 安装](https://github.com/creationix/nvm)

nvm 安装 node 和管理 node。

```bash
$ nvm ls-remote         // 列出可以安装的 node 版本号
$ nvm install v8.10.0    // 安装指定版本
$ nvm uninstall v8.10.0  // 删除指定版本
$ nvm use v8.10.0       // 切换 node 版本
$ nvm current          // 查看当前版本
$ nvm ls              // 查看已经安装的版本
```

## 二、NRM

Node 经常遇到的一个问题就是安装包太慢，官网提供的镜像资源服务器在国外，所以经常吐槽官网的镜像。

**nvm 安装**

```bash
$ npm install -g nrm
```

列出可用的镜像源

```bash
$ nrm ls
```

会列出下面的几个源，带 * 号的表示当前的采用的镜像源。

```
  npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
* taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
```

如果想使用其他的源，例如想换成 cnpm 的源

```bash
$ nrm use cnpm
```

会提示
```
Registry has been set to: http://r.cnpmjs.org/
```

## 三、淘宝镜像源和 CNPM

直接参考这篇文章，不复制粘贴[淘宝镜像源和 CNPM](https://npm.taobao.org/)

## 四、NPM

直接参考官网文档，介绍的最权威[官网文档](https://www.npmjs.com.cn/)，必读，必须熟悉，少废话。

## 五、Package.json

同样给两个地址自己读，同样是 Node 中非常重要的必须要熟悉的知识

> - [package.json](https://www.npmjs.com.cn/files/package.json/)
> - [Working with package.json](https://www.npmjs.com.cn/getting-started/using-a-package.json/)



