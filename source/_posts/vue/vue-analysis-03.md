---
layout: post
title: Vue 源码分析（三）：源码构建
date: 2018-06-01 22:39:05
tags: Vue
categories: JavaScript
---

Vue.js 源码是基于 Rollup 构建的，它的构建相关配置都在 scripts 目录下。

构建的入口 JS 文件，在 `scripts/build.js` 中，Vue.js 源码构建的脚本如下

```js
{
  "script": {
    "build": "node scripts/build.js",
    "build:ssr": "npm run build -- web-runtime-cjs,web-server-renderer",
    "build:weex": "npm run build --weex"
  }
}
```

scripts/config.js 注释

```js
const path = require('path')
const buble = require('rollup-plugin-buble')
const alias = require('rollup-plugin-alias')
const cjs = require('rollup-plugin-commonjs')
const replace = require('rollup-plugin-replace')
const node = require('rollup-plugin-node-resolve')
const flow = require('rollup-plugin-flow-no-whitespace')
const version = process.env.VERSION || require('../package.json').version
const weexVersion = process.env.WEEX_VERSION || require('../packages/weex-vue-framework/package.json').version

// 每个文件头部注释
const banner =
  '/*!\n' +
  ' * Vue.js v' + version + '\n' +
  ' * (c) 2014-' + new Date().getFullYear() + ' Evan You\n' +
  ' * Released under the MIT License.\n' +
  ' */'

const weexFactoryPlugin = {
  intro () {
    return 'module.exports = function weexFactory (exports, document) {'
  },
  outro () {
    return '}'
  }
}

const aliases = require('./alias')
/**
 * @param {*} p
 */
const resolve = p => {
  /**
   * 1. 例如 web/entry-runtime.js， base 就是 'web'
   * 2. 通过 alias 文件，aliases['web'] 获取到 resolve('src/platforms/web') 的文件地址
   * 3. p.slice(base.length + 1) 返回 'entry-runtime.js'
   */
  const base = p.split('/')[0]
  if (aliases[base]) {
    /**
     * 例如
     * path.resolve(aliases[base], p.slice(base.length + 1)) 返回 /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-runtime.js
     * aliases[base] 返回 /Users/tyrant/Workspace/Vue/vue/src/platforms/web
     * p.slice(base.length + 1) 返回 entry-runtime.js
     */
    /**
     * 以下返回
     * /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-runtime.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-runtime-with-compiler.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-runtime.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-runtime-with-compiler.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-runtime.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-runtime.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-runtime-with-compiler.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-runtime-with-compiler.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-compiler.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-compiler.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-server-renderer.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-server-basic-renderer.js
      /Users/tyrant/Workspace/Vue/vue/src/server/webpack-plugin/server.js
      /Users/tyrant/Workspace/Vue/vue/src/server/webpack-plugin/client.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/weex/entry-runtime-factory.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/weex/entry-framework.js
      /Users/tyrant/Workspace/Vue/vue/src/platforms/weex/entry-compiler.js
    */
    return path.resolve(aliases[base], p.slice(base.length + 1))
  } else {
    /**
     * 以下返回
     * /Users/tyrant/Workspace/Vue/vue/dist/vue.runtime.common.js
      /Users/tyrant/Workspace/Vue/vue/dist/vue.common.js
      /Users/tyrant/Workspace/Vue/vue/dist/vue.runtime.esm.js
      /Users/tyrant/Workspace/Vue/vue/dist/vue.esm.js
      /Users/tyrant/Workspace/Vue/vue/dist/vue.runtime.js
      /Users/tyrant/Workspace/Vue/vue/dist/vue.runtime.min.js
      /Users/tyrant/Workspace/Vue/vue/dist/vue.js
      /Users/tyrant/Workspace/Vue/vue/dist/vue.min.js
      /Users/tyrant/Workspace/Vue/vue/packages/vue-template-compiler/build.js
      /Users/tyrant/Workspace/Vue/vue/packages/vue-template-compiler/browser.js
      /Users/tyrant/Workspace/Vue/vue/packages/vue-server-renderer/build.js
      /Users/tyrant/Workspace/Vue/vue/packages/vue-server-renderer/basic.js
      /Users/tyrant/Workspace/Vue/vue/packages/vue-server-renderer/server-plugin.js
      /Users/tyrant/Workspace/Vue/vue/packages/vue-server-renderer/client-plugin.js
      /Users/tyrant/Workspace/Vue/vue/packages/weex-vue-framework/factory.js
      /Users/tyrant/Workspace/Vue/vue/packages/weex-vue-framework/index.js
      /Users/tyrant/Workspace/Vue/vue/packages/weex-template-compiler/build.js
     */
    return path.resolve(__dirname, '../', p)
  }
}

const builds = {
  // Runtime only (CommonJS). Used by bundlers e.g. Webpack & Browserify
  // 不同版本的 vue 的编译配置
  'web-runtime-cjs': {
    entry: resolve('web/entry-runtime.js'), // 入口 'src/platforms/web/entry-runtime.js'
    dest: resolve('dist/vue.runtime.common.js'), // 编译输出
    format: 'cjs',    // 构建出来的文件格式 commonJS?
    banner
  },
  // Runtime+compiler CommonJS build (CommonJS)
  'web-full-cjs': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.common.js'),
    format: 'cjs',
    alias: { he: './entity-decoder' },
    banner
  },
  // Runtime only (ES Modules). Used by bundlers that support ES Modules,
  // e.g. Rollup & Webpack 2
  'web-runtime-esm': {
    entry: resolve('web/entry-runtime.js'),
    dest: resolve('dist/vue.runtime.esm.js'),
    format: 'es',   // es6 格式
    banner
  },
  // Runtime+compiler CommonJS build (ES Modules)
  'web-full-esm': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.esm.js'),
    format: 'es',
    alias: { he: './entity-decoder' },
    banner
  },
  // runtime-only build (Browser)
  'web-runtime-dev': {
    entry: resolve('web/entry-runtime.js'),
    dest: resolve('dist/vue.runtime.js'),
    format: 'umd',
    env: 'development',
    banner
  },
  // runtime-only production build (Browser)
  'web-runtime-prod': {
    entry: resolve('web/entry-runtime.js'),
    dest: resolve('dist/vue.runtime.min.js'),
    format: 'umd',
    env: 'production',
    banner
  },
  // Runtime+compiler development build (Browser)
  'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',    // UMD 格式
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
  },
  // Runtime+compiler production build  (Browser)
  'web-full-prod': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.min.js'),
    format: 'umd',
    env: 'production',
    alias: { he: './entity-decoder' },
    banner
  },
  // Web compiler (CommonJS).
  'web-compiler': {
    entry: resolve('web/entry-compiler.js'),
    dest: resolve('packages/vue-template-compiler/build.js'),
    format: 'cjs',
    external: Object.keys(require('../packages/vue-template-compiler/package.json').dependencies)
  },
  // Web compiler (UMD for in-browser use).
  'web-compiler-browser': {
    entry: resolve('web/entry-compiler.js'),
    dest: resolve('packages/vue-template-compiler/browser.js'),
    format: 'umd',
    env: 'development',
    moduleName: 'VueTemplateCompiler',
    plugins: [node(), cjs()]
  },
  // Web server renderer (CommonJS).
  'web-server-renderer': {
    entry: resolve('web/entry-server-renderer.js'),
    dest: resolve('packages/vue-server-renderer/build.js'),
    format: 'cjs',
    external: Object.keys(require('../packages/vue-server-renderer/package.json').dependencies)
  },
  'web-server-renderer-basic': {
    entry: resolve('web/entry-server-basic-renderer.js'),
    dest: resolve('packages/vue-server-renderer/basic.js'),
    format: 'umd',
    env: 'development',
    moduleName: 'renderVueComponentToString',
    plugins: [node(), cjs()]
  },
  'web-server-renderer-webpack-server-plugin': {
    entry: resolve('server/webpack-plugin/server.js'),
    dest: resolve('packages/vue-server-renderer/server-plugin.js'),
    format: 'cjs',
    external: Object.keys(require('../packages/vue-server-renderer/package.json').dependencies)
  },
  'web-server-renderer-webpack-client-plugin': {
    entry: resolve('server/webpack-plugin/client.js'),
    dest: resolve('packages/vue-server-renderer/client-plugin.js'),
    format: 'cjs',
    external: Object.keys(require('../packages/vue-server-renderer/package.json').dependencies)
  },
  // Weex runtime factory
  'weex-factory': {
    weex: true,
    entry: resolve('weex/entry-runtime-factory.js'),
    dest: resolve('packages/weex-vue-framework/factory.js'),
    format: 'cjs',
    plugins: [weexFactoryPlugin]
  },
  // Weex runtime framework (CommonJS).
  'weex-framework': {
    weex: true,
    entry: resolve('weex/entry-framework.js'),
    dest: resolve('packages/weex-vue-framework/index.js'),
    format: 'cjs'
  },
  // Weex compiler (CommonJS). Used by Weex's Webpack loader.
  'weex-compiler': {
    weex: true,
    entry: resolve('weex/entry-compiler.js'),
    dest: resolve('packages/weex-template-compiler/build.js'),
    format: 'cjs',
    external: Object.keys(require('../packages/weex-template-compiler/package.json').dependencies)
  }
}

function genConfig (name) {
  const opts = builds[name]
  // rollup 打包配置结构
  const config = {
    input: opts.entry,
    external: opts.external,
    plugins: [
      replace({
        __WEEX__: !!opts.weex,
        __WEEX_VERSION__: weexVersion,
        __VERSION__: version
      }),
      flow(),
      buble(),
      alias(Object.assign({}, aliases, opts.alias))
    ].concat(opts.plugins || []),
    output: {
      file: opts.dest,
      format: opts.format,
      banner: opts.banner,
      name: opts.moduleName || 'Vue'
    }
  }

  if (opts.env) {
    config.plugins.push(replace({
      'process.env.NODE_ENV': JSON.stringify(opts.env)
    }))
  }

  Object.defineProperty(config, '_name', {
    enumerable: false,
    value: name
  })

  return config
}

if (process.env.TARGET) {
  module.exports = genConfig(process.env.TARGET)
} else {
  exports.getBuild = genConfig
  // console.log('getAllBuilds', Object.keys(builds).map(genConfig))
  /**
   * getAllBuilds 打印出以下rollup 打包结构
   * {
      input: '/Users/tyrant/Workspace/Vue/vue/src/platforms/web/entry-server-renderer.js',
      external: ['chalk',
        'hash-sum',
        'he',
        'lodash.template',
        'lodash.uniq',
        'resolve',
        'serialize-javascript',
        'source-map'
      ],
      plugins: [
        [Object],
        [Object],
        [Object],
        [Object]
      ],
      output: {
        file: '/Users/tyrant/Workspace/Vue/vue/packages/vue-server-renderer/build.js',
        format: 'cjs',
        banner: undefined,
        name: 'Vue'
      }
    }
   */
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}

```











