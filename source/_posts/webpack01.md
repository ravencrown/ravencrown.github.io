---
layout: post
title: Webpack 入门（一）
date: 2018-08-05 16:30:36
tags: Webpack
categories: JavaScript
---

`webpack` 是一个现代 `JavaScript` 应用程序的静态模块打包器(`module bundler`)
在打包过程中，会从入口开始递归地构建一个依赖关系图，其中包含应用程序需要用到的每个模块，
然后将这些模块打包成一个或多个`bundle`

webpack 处理前端的资源文件：JS、CSS、图片，将源文件转成浏览器能识别的

ES6 -- (loader/plugin) --> ES5
SASS/LESS/Stylus -- (loader/plugin) --> CSS
image -- (loader/plugin) --> 压缩、转base64格式
Coffee/ts -- (loader/plugin) --> js
ETC...

## 四个核心概念

- entry 入口源文件
- 输出 生成文件
- loader 编译文件
- plugin 比loader更强大，能使用更多webpack的api

### 入口 Entry

入口起点(`entry`)指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。每个依赖项随即被处理，最后输出到称之为 `bundles` 的文件中

简单说：就是 HTML 直接引用的 JS 文件，其他非入口文件则作为直接或间接的依赖引入

**用法**
```js
// entry: string|Array<string>
// 传数组的时候，可以将多个依赖文件导向一个 chunk，打包到一个 bundle 文件，参考 CommonsChunkPlugin 插件

// 单页面
const config = {
  entry: './path/to/my/entry/file.js'
};


// 多页面：根据经验每个页面一个入口
const config = {
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }
};
```

### 输出 output

打包之后输出的配置，只能有一个

示例
```js
output: {
    path: path.resolve(__dirname + '/dist'),  // webpack3 之后使用绝对路径
    filename: 'js/[name]-[chunkhash].js',
    publicPath: 'http://cdn.com/'
},

```

**filename 占位符**

- name 模块名称。 例上面入口的 pageOne，pageTwo，pageThree
- hash 模块编译后的（整体）Hash值
- chunkhash 分片的Hash值（每个模块的hash）

<img width="792" alt="d28ab5fd-b360-4185-9ba5-8d4a1b16c92a" src="https://user-images.githubusercontent.com/9835391/34867635-d818c624-f7bb-11e7-9e16-546f9f84efd0.png">



## 核心概念：Loader

loader 它是指用来将一段代码转换成另一段代码。loader 可以使你在 import 或"加载"模块时预处理文件。因此，loader 类似于其他构建工具中“任务(task)”

例如：比如在 `require('a.coffee')` 的时候，webpack 自动先编译成 JS 文件，再引入进来

类似 gulp 中的 task 任务

### loader的特性

- loader 可以是串联方式的，例如 `style-loader!css-loader`
- loader 可以是同步也可以是异步的
- loader 可以运行在 Node.js 中，并且可以执行任何操作
- loader 可以接受查询参数（query）
- loader 在命令行中是通过 --module-bind（webpack -h） 的方式绑定，也可以在配置文件中通过正则表达式指定某个后缀名的文件由哪个 loader 来执行

- loader 可以通过 NPM 安装/发布
- loader 可以获取到 webpack 的配置
- plugin 可以给予 loader 更多的特性
- loader 可以产生一个额外的任意文件

### loader 的使用

三种方式

1. 在 require 或者 import 声明中指定

```js
import Styles from 'style-loader!css-loader?modules!./styles.css';
//or
require("style-loader!css-loader?modules!./styles.css");
```

2. 通过配置文件

```js
 module: {
    rules: [
      {
        // test 是对资源的正则匹配，如果匹配到了，就应用下面的 loader 对匹配到的资源进行处理
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          }
        ]
      }
    ]
  }
```

3. 通过命令行 CLI

```js
// webpack 执行一次开发时的编译
// webpack -p 执行一次生成环境的编译（压缩）
// webpack --watch 在开发时持续监控增量编译（很快）
// webpack -d 让他生成SourceMaps
// webpack --progress 显示编译进度
// webpack --colors 加个颜色，变漂亮
// webpack --sort-modules-by, --sort-chunks-by, --sort-assets-by 将modules/chunks/assets进行列表排序
// webpack --display-chunks 展示编译后的分块
// webpack --display-reasons 显示更多引用模块原因
// webapck --display-error-details 显示更多报错信息

$ webpack --module-bind jade-loader --module-bind css=style-loader!css-loader
```

## 使用 babel-loader 转换 ES6 代码

安装 babel 参考 http://babeljs.io/docs/setup/#installation

### 安装

```s
$ npm install --save-dev babel-loader babel-core
```

### 使用方式

```js
// Via config
module: {
  rules: [
    { 
        test: /\.js$/, 
        exclude: /node_modules/, 
        loader: "babel-loader" 
    }
  ]
}

// Via loader
var Person = require("babel!./Person.js").default;
new Person();
```

### 配置文件 `.babelrc`

**在 `.babelrc` 文件中定义**
```js
{
  "presets": ["env"]
}
```

### presets 是干什么用的？ 

设置转码规则，版本参考 https://babeljs.cn/docs/plugins/

转换对应模块参考图
![6147470b-73c6-4ba1-96d6-3c536beff662](https://user-images.githubusercontent.com/9835391/34867649-eafcdadc-f7bb-11e7-92b0-512151410ee4.png)

例如
`const和let现在一律转换成var`
```js
// 转换前
var a = 1;
let b = 2;
const c = 3;

//转换后
var a = 1;
var b = 2;
var c = 3;
```

`let 的块级作用域如何体现`
```js
// 转换前
let a1 = 1;
let a2 = 6;

{
    let a1 = 2;
    let a2 = 5;

    {
        let a1 = 4;
        let a2 = 5;
    }
}
a1 = 3;

// 转换后
var a1 = 1;
var a2 = 6;

{
    var _a1 = 2;
    var _a2 = 5;

    {
        var _a3 = 4;
        var _a4 = 5;
    }
}
a1 = 3;

// 实质就是在块级作用域改变一下变量名，使之与外层不同
```

`赋值解构转换`
```js
// 转换前
var props = {
    name: "heyli",
    getName: function() {

    },
    setName: function() {

    }
};
let { name, getName, setName } = this.props;

// 转换后
var props = {
    name: "heyli",
    getName: function getName() {},
    setName: function setName() {}
};

var name = props.name;
var getName = props.getName;
var setName = props.setName;
```

`扩展运算符`
```js
// 转换前
// ...y代表它接收了剩下的参数。也就是arguments除了第一个标号的参数之外剩余的参数。
function func(x, ...y) {
    console.log(x);
    console.log(y);
    return x * y.length;
}

// 转换后
function func(x) {
    console.log(x);

    for (var _len = arguments.length, y = Array(_len > 1 ? _len - 1 : 0), _key = 1; _key < _len; _key++) {
        y[_key - 1] = arguments[_key];
    }

    console.log(y);
    return x * y.length;
}
```


**配置 presets 的三种方式**

1. 在webpack 配置文件 loader 中直接配置
```js
{
    test: /\.js$/,
    loader: 'babel',
    query: {
        presets: ['lastest']
    }
}
```

2. `.babelrc` 配置文件中配置
```js
{
    presets: ['lastest']
}
```

3. 在 package.json 中配置

```json
"babel": {
    "presets": ["lastest"]
}
```

### 常用的 Loader

- 处理样式：`less-loader, sass-loader, style-loader, postcss-loader`
- 图片处理：`url-loader, file-loader`
- js: `babel-loader，babel-preset-es2015，babel-preset-react` 等等
- 将js模块暴露到全局：`expose-loader`


### module.loaders

module.loaders 可配置的参数

- test: 正则匹配方式
- exclude: loader 处理的资源的排除的范围
- include: loader 处理包含的范围，可以是正则、数组、绝对路径
- loader: 使用 "!" 分割 loader，从后往前处理
- loaders: 以 loader 组成的 数组

```js
loaders: [
    {
        test: /\.js$/,
        loader: 'babel-loader',
        // 指定 exclude，include 可以提升打包速度
        exclude: path.resolve(__dirname, '/node_modules/'),
        include: path.resolve(__dirname, '/src/'),
        query: {
            "presets": ["latest"]
        }
    },
    {
        test: /\.css$/,
        loader: 'style-loader!css-loader!postcss-loader',
        loaders: [
            'style-loader',
            'css-loader',
            'postcss-loader'
        ]
    }
]
```


## webpack 处理项目中的 CSS 文件

- css-loader: 支持 css 文件作为模块引入 js 中（require/import） 方式
- style-loader: 为了在html中以style的方式嵌入css

```js
{
    test: /\.css$/,
    // 处理顺序，从右到左
    // importLoaders=1 的作用
    loader: 'style-loader!css-loader?importLoaders=1'
},
{
    test: /\.css$/,
    loader: 'postcss-loader',
    options: {
        plugins() {
            return [
                require('autoprefixer')({
                    browsers: [
                        '>1%',
                        'last 2 versions',
                        'Firefox ESR',
                        'not ie < 9'
                    ]
                })
            ]
        }
    }
}
```

## webpack 处理项目中的 LESS/SASS 文件

less-loader 
sass-loader

# webpack 处理项目中的模板文件

html-loader 看图

![23222d21-dc2a-40a5-8462-344bb48d8ac3](https://user-images.githubusercontent.com/9835391/34867665-f72bc8d6-f7bb-11e7-9e57-8827b2034043.png)


ejs-loader

https://doc.webpack-china.org/loaders/    `模板(Templating)`

## 核心概念：Plugins

加强版 Loader,可以使用 webpack 的API，需要通过require()引入并将插件实例添加到plugins数组中
如果还记得，可以介绍下 html-webpack-plugin

- 代码热替换, HotModuleReplacementPlugin (开发的时候无需刷新页面即可看到更新)
- 生成 html 文件，HtmlWebpackPlugin
- 将 CSS 成生文件，而非内联，ExtractTextPlugin
- 代码丑化，UglifyJsPlugin，开发过程中不建议打开
- 多个 html共用一个 js 文件 (chunk)，可用 CommonsChunkPlugin,
- webpack-bundle-analyzer: 用视图的形式分析bundle.js中的每个模块
- 清理文件夹，Clean
- 报错但不退出webpack进程，NoErrorsPlugin
- 调用模块的别名 ProvidePlugin，例如想在 js 中用 $，如果通过 webpack 加载，需要将 $ 与 jQuery 对应起来


<img width="780" alt="55cdd3e7-c74c-471f-b072-b7d96e9a5f5a" src="https://user-images.githubusercontent.com/9835391/34867687-08ec271e-f7bc-11e7-976e-afec483c62a0.png">


![d2efe2b5-97d8-4fc5-b4b3-f5b221336f21](https://user-images.githubusercontent.com/9835391/34867723-2501ee7a-f7bc-11e7-95c8-c2d09440145a.png)


## 参考资料 

- [阮一峰 Webpack Demos](https://github.com/ruanyf/webpack-demos) △
- [postcss github](https://github.com/postcss/postcss)
- [参考 babel 官网](http://babeljs.io/docs/plugins/)
- [阮一峰 Babel 入门教程](http://www.ruanyifeng.com/blog/2016/01/babel.html)
- [如何编写一个Plugin](https://doc.webpack-china.org/contribute/writing-a-plugin/#-)
- [如何编写一个Loader](https://doc.webpack-china.org/contribute/writing-a-loader/#-)



