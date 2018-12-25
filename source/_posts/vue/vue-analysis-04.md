---
layout: post
title: Vue 源码分析（四）：入口 entry-runtime-with-compiler.js 解析
date: 2018-06-04 12:29:05
tags: Vue
categories: JavaScript
---


我们之前提到过 Vue.js 构建过程，在 web 应用下，我们来分析 Runtime + Compiler 构建出来的 Vue.js，它的入口是 `src/platforms/web/entry-runtime-with-compiler.js`

那么，当我们的代码执行 `import Vue from 'vue'` 的时候，就是从这个入口执行代码来初始化 Vue， 那么 Vue 到底是什么，它是怎么初始化的，我们来一探究竟。

```js
import config from 'core/config'
import { warn, cached } from 'core/util/index'
import { mark, measure } from 'core/util/perf'

import Vue from './runtime/index'
import { query } from './util/index'
import { compileToFunctions } from './compiler/index'
import { shouldDecodeNewlines, shouldDecodeNewlinesForHref } from './util/compat'
...
```

上面代码可以看到 Vue 来自 `'./runtime/index'`，我们再看`'./runtime/index'`,

```js
import Vue from 'core/index'
import config from 'core/config'
import { extend, noop } from 'shared/util'
import { mountComponent } from 'core/instance/lifecycle'
import { devtools, inBrowser, isChrome } from 'core/util/index'
...
```

同样显示 Vue 来自 `'core/index'`, 那我们继续看 `'core/index'`，

```js
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'

// 关键代码 - 初始化全局 Vue API
initGlobalAPI(Vue)
...
```

同样显示 Vue 来自 `'instance/index'`, 那我们继续看 `'instance/index'`，

```js
/**
 * 本文件总结：Vue 是用 function 来实现的一个类（Vue 就是一个类），类上挂了很多原型的方法
 * 其他的文件 runtime/index.js、包括入口entry-runtime-with-compiler.js等文件也会挂载原型方法
 * 公共全局放在定义放在 core/global-api 文件夹下
 */
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

// Vue 的庐山真面目，其实就是一个 function（为什么通过 function 实现 Vue？而不是通过 class?）
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    // Vue 必须要通过 new 的方法去实例化（es5 实现 class 的方式）
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
 
// 这里的很多 mixin 方法、都是在往 Vue.prototype 上挂载方法（es6往原型上挂载方法不方便，es6的原型继承已经被修改过）
// 在不同的文件定义原型上的方法，细致化
initMixin(Vue) // 在 vue 原型上挂了个方法_init，Vue.prototype._init
stateMixin(Vue) // 挂载 $set、$delete、$watch 等方法
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

在这里，我们终于看到了 Vue 的庐山真面目，它实际上就是一个用 Function 实现的类，我们只能通过 new Vue 去实例化它。

有些同学看到这不禁想问，为何 Vue 不用 ES6 的 Class 去实现呢？我们往后看这里有很多 xxxMixin 的函数调用，并把 Vue 当参数传入，它们的功能都是给 Vue 的 prototype 上扩展一些方法（这里具体的细节会在之后的文章介绍，这里不展开），Vue 按功能把这些扩展分散到多个模块中去实现，而不是在一个模块里实现所有，这种方式是用 Class 难以实现的。


## initGlobalAPI

Vue.js 在整个初始化过程中，除了给它的原型 prototype 上扩展方法，还会给 Vue 这个对象本身扩展全局的静态方法，它的定义在 src/core/global-api/index.js 中

```js
export function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  // 定义 config
  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  /**
   * Vue util，这里的实现不稳定，可能会变
   */
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
  }

  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  // 
  Vue.options._base = Vue

  // 把 Vue 的内置组件 keep-alive 扩展到 Vue.options.components 上
  extend(Vue.options.components, builtInComponents)

  initUse(Vue) // 定义 Vue.use 的全局方法
  initMixin(Vue)  // 定义 Vue.mixin 的全局方法
  initExtend(Vue) // 定义 Vue.extend 的全局方法 
  initAssetRegisters(Vue) // 定义 Vue.component、Vue.directive、Vue.filter
}
```

这里就是在 Vue 上扩展的一些全局方法的定义，Vue 官网中关于全局 API 都可以在这里找到，这里不会介绍细节，会在之后的章节我们具体介绍到某个 API 的时候会详细介绍。有一点要注意的是，Vue.util 暴露的方法最好不要依赖，因为它可能经常会发生变化，是不稳定的。

## shared/constants.js 定义

```js
export const SSR_ATTR = 'data-server-rendered'

export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]

// 生命周期钩子
export const LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured'
]
```

## 内置组件 keep-alive

```js
core/components/index.js
```

