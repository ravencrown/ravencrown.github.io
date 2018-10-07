---
layout: post
title: Live - 帮助你深入理解 React
date: 2018-08-19 12:51:34
tags: React
categories: React
---

本笔记是知乎程墨的《帮助你深入理解React》 Live 的笔记

课程主要是介绍 React 的

> - 深入理解 React 工作原理
> - JSX 优势和局限
> - Props 和 State 的用法
> - React 生命周期
> - 为什么尽量使用无状态组件
> - 高阶组件
> - 组件之间的通信


## 一、理解React的工作原理

React 本质有三个主要的方面

> 1. UI = f(data)
> 2. 一切都是组件
> 3. 声明式变成（Declarative Programming)


** （1）UI = f(data) **

改变UI的作用，实际上是通过改变 data, 让 data 驱动 function, function 影响 UI 的渲染。这个 data 实际上就是我们所说的 Props 和 State

**（2） virtual-dom **

> 1. 传统diff 算法背后原理, 类似git diff的功能，传统的 diff 时间复杂度为 O(n2)
> 2. React 的 diff 原理时间复杂度在 O(n), 采用 mount 和 unmount 的过程。例如图一, 当 A 节点移动的时候，A 节点有 unmount 和 mount 的过程，在 React 里面，不认为 A移动了，只认为 A 废掉了， unmount了，然后在 B 节点上重新 mount 过来，这样时间复杂度只在 O(n)。原理就是从根节点开始比对，然后以递归的方式逐步的往下比对。

![图一](https://pic2.zhimg.com/v2-bebdaf74d8718a2ce99cec7b45d36d2b_b.jpg)

例如，现在有两个节点要比对，第一步先看两个节点的类型是否一样，如果类型不相同，一个是 div, 一个是 p,那就直接认为彻底改变了，有 diff 的情况，然后节点就放弃掉，执行 unmount 和 mount。如果类型相同，那就根据 update 的生命周期来判断

现在有个例子，如图二，R 节点上有 ABC 三个组件，类型都一样，现在中间插了 X 的组件，这时候需要开发人员协助，因为 React 完全不知道 ABC 和 X 之间有什么关系。这个时候每个组件需要一个 key, 例如 ABC 的 key 分别是 abc, 新组件 X 的 key 是 x, 这样 React 根据 key 的对比知道只不过是加了一个新的组件 X。

**key 的要素：唯一、稳定的（组件 A 的 key 整个过程的 key 是不变的）**

动态子组件是由一个数组来产生的。所以有很多开发人员经常用数组的下标作为 key，但是这样 key 就不稳定了。

![图二](https://pic2.zhimg.com/v2-ff242173690d56f59d3f2f22ed4d8f3a_b.jpg)

三元运算符组件是否需要 key, 如以下代码

```html
<div>
    <p>hello</p>
    {
        someCondition ? <p>what?</p> : null
    }
    <p>world</p>
</div>
```

答案是不需要，三元运算符不需要加 key, 只有数组才需要加 key

如图三

![图三](https://pic2.zhimg.com/v2-a00f34c33ca9989cc0cecc6ffeb47833_r.jpg)


** （3）React中一切皆为组件(Component) **



** （4）声明式编程 （Declarative Programming) **

大致意思是不会根据 React 的 API 的改动修改 Code, 不像 jQuery 一样，需要根据 API 的更新改动原来写好的代码，例如以前用的是 delegate，之后 API 更新了，有了 on，原来的代码就要把所有的 delegate 更新成 on。jQuery 的编程是命名式编程。React 的代码不太可能主动去调用系统级的 API，相反是实现函数，例如 render，componentDidMount等等，这样子 React 的实现方式可能会改变，但是不需要改变 code。

## 二、JSX 的优势与局限

render函数是个纯函数，不做任何直接渲染的事情。只是返回了一些指令，由React对这些指令做真正DOM操作。

** （1）JSX 在线转义**

例如

```js
JSX:
<button type="submit">
    Save
</button>
```

上面这段代码，babel 转义之后变成下面这段代码

```js
Transpiled JS:
React.createElement(
    "button",
    { type: "submit" },
    "Save"
);
```

React.createElement返回的是一个对DOM的描述，并不是真正去修改DOM。然后由React对描述做Virtual DOM的比对和修改动作。实际上你可以不用JSX定义自己的一套函数

可以看看下面的这段伪代码

```js
// h 函数可以看做 React.createElement
h('div.someclass', [
    h('p', 'Child One'),
    h('p', {onClick: () => console.log('Got click.')}, 'Child Two'),
    h(SomeComponent, 'Child Three'),
    h('p', {className: 'anotherClass', key: 'four'}, 'Child Four'),
]);
```

**render 函数到目前为止只能返回一个 JSX，或者 null/undefined，但是在 v16以后，render 函数可以返回一个数组**

JSX 可以包含花括号，花括号里面最后都是一个 JavaScript 表达式，实际上都是 createElement 的某个参数。正因为它是参数，所以它必须是表达式，它不能是语句。故不可能有for、while循环，因为二者一出现，就表示这是语句，不是表达式。

可以看看以下转义前和转义后的代码

```js
JSX：
<div>
  {
    arr.map(x => <div>x</div> )
  }
</div>
```

<br/>

```js
Transpiled JavaScript:
React.createElement(
  "div",
  null,
  arr.map(x => React.createElement(
    "div",
    null,
    "x"
  ))
);
```

可以对比 arr.maps 里面的参数。`x => <div>x</div>` 是一个参数，所以不能是语句，只能是表达式。在这里表达式可以是 map/reduce/filter/concat 这些函数，也可以使用三元云算法。

在 React 的 render 函数，不要使用 push/reverse 这种数组的函数。因为 render 函数应该是纯函数，它不应该有任何副作用。渲染的东西最后是 data，这个 data 可能是 state 或者 props。如果用了 push 或者 reverse，实际上改的是 props 或者 state，最后产生的结果不可预料，因为你不是纯函数。

## 三、使用props还是state

props 指外部传入的数据，state 是组件内部的一个状态。但一个组件它自己的 state 可以作为传递给它子组件的props的一个数据来源。如图四

一个组件要改变自己的状态，怎么做呢？只能通过 setState 改变自己的状态。一个组件绝对不可能说修改自己的 props 来引发自己的更新状态，传递过来的 props 不应该被修改。

![图四](https://pic2.zhimg.com/v2-5ca6f25ea1ceddeea7bd57ac492193e7_b.jpg)

尽量应该让一个组件做的事情少一点，让它state尽量少，所以提倡一个没有state的Component。但凡props能搞定的事情，就不要牵扯到state，最好把state往外堆，堆到系统边缘的地方。

`问：我将父级所传递的props，作为子级的state，是否会产生值引用对象影响问题？也就是我更改子级state，父级props被相应更改`

**答：会产生引用对象问题。因为React不会做深度拷贝事情。结果不可预知。**

## 四、详解React组件的生命周期

三种过程 

> 1. mount - 组件从无到有的过程
> 2. update - 又分为state change引发的和props引发的。state/props 改变之后组件重新绘制一遍
    - 组件内部自己的 state 导致的重绘
    - 父组件传给它的 props 改变引发的
> 3. unmount - 组件从有到无

** （1）mount过程：**

> 1. getDefaultProps
> 2. getInitialState
> 3. componentWillMount
> 4. render
> 5. componentDidMount (只在浏览器端执行，如果在服务器端做 react 渲染，这个是不会执行的，因为服务器端最后吐出来的是一个字符串，不存在 DOM tree节点被 mount 的过程)

** （2）update 过程**

因state改变引发的update过程：

> 1. shouldComponentUpdate
> 2. componentWillUpdate
> 3. render
> 4. componentDidUpdate

因父组件想要render这个组件改变引发的update过程：

> 1. componentWillReceiveProps
> 2. shouldComponentUpdate
> 3. componentWillUpdate
> 4. render
> 5. componentDidUpdate

由于父组件如果想重绘子组件，不管传过来的 props 是否变化，只要重绘，都会执行上面过程。shouldComponentUpdate 此时就很重要，它可以中间截胡，通过返回 false 就可以让后面的三个函数不会被执行，这就节省了很多不必要的渲染时间。

以 render 为界，不管是 mount 过程还是 update 过程，render 之前所有函数被调用时，这些组件的 state 和 props 都是没有被改变的。在 render 之前的函数里访问 this.props 或者 this.state 都是未改变之前的。只有当 render 函数开始执行的时候，state/props 才是更新之后的值

在 V16 里，一个组件的渲染过程是可以被打断的，回头再做。在 V16 之后，以 render 函数为界，前面所有的函数可能会被执行很多次。只要开始执行 render 函数，state 和 props 就被改变了，就不能再被打断了，会一直往下调。

所以 render 函数之前的这些函数一定要写成纯函数，如果写的函数不是纯函数，有副作用，若被执行两次，就会产生意想不到的结果。所以今后写代码，这些函数要尽量写成纯函数。

## 五、为什么尽量构建无状态组件？

如图五

![图五](https://camo.githubusercontent.com/2384d434bcdaac44f4f94c22a7c4c66632be91ad/68747470733a2f2f706963332e7a68696d672e636f6d2f76322d38663737386165323862643533366161636461666339306230643937383038645f722e6a7067)


图五为一个假象的组件树结构，蓝色的是无状态组件。维护状态的组件往上推，推到靠近 root 的地方去。尽量让少数的组件去管理状态，让他有 state, 然后有 state 的组件作为父组件存在，它所控制的子组件，都是无状态的。

需注意 React 世界不是只有两种有状态和无状态组件，这种分类比较粗。有的组件比较特殊，如下每隔多少时间发送一个心跳信息，这是一个副作用的事情。

```js
import React from 'react';

export default class HeartBeat extends React.Component {
  render() {
    return null;   //意思什么都不画。
  }

  componentDidMount() {
    this.timer = setInterval(() => {
      fetch('/api/v1/heartbeat');
    }, 5000);
  }

  componentWillUnmount() {
    clearInterval(this.timer);
  }
}
```


## 六、创建高阶组件（HoC，Higher-Order Component)

但凡发现一个逻辑在两个组件里面用，都可以把它抽象成高阶组件。高阶组件其实是一个函数，它接收一个或者多个参数(参数往往是组件)，返回一个全新的组件。

如以下的代码

```js
// 包裹方式
const HoC = (WrappedComponent) => {
    const WrappingComponent = (props) => (
        <div className="foo">
            <WrappedCompoent {...props} />
        </div>
    );
    return WrappingComponent;
};
```
<br/>

```js
// 继承方式
const HoC = (WrappedComponent) => {
    class WrappingComponent extends WrappendComponent {
        render() (
            const {user, ...otherProps} = this.props;
            this.props = otherProps;
            return super.render();
        )
    }
    return WrappingComponent;
};
```

继承的方式：不得不通过 super.render() 等函数调用父级的生命周期函数，这种方式很不好，它将两个组件纠缠在一起，会有共享函数、共享的属性，这种方式比较违背组件的原则。所以尽量不要采用继承的方式实现 HOC。


** （1）HOC误解**

> - HOC可以有多个参数，不是只能有一个参数
> - HOC可以有多个参数表示多个组件，不是只能有一个参数作为组件类

```js
// 用户登录，正常登录组件和提示用户登录的组件 
const HoC = (WrappedComponent, LoginView) => {
    const WrappingComponent = () => {
        const {user} = this.props;  
        if (user) {
            return <WrappedComponent {...this.props} />
        } else {
            return <LoginView {...this.props} />
        }
    };
    return WrappingComponent;
};
```

** （2）HOC常见应用场景有哪些？**

有一些功能，可以用在不同的组件类，但是不想重复代码，这个时候就需要HOC。

## 七、组件之间通讯

** （1）组件通信的三种方式**

** （2）父子通信**

如图六

> - 父组件通过props传递给子组件。子组件想传递给父组件，调用父组件的方法。
> - 调用ref(不推荐)。子组件有个foo方法，父组件调用ref.foo来执行，这种方式不太好。
> - 可以用callback或者promise。父组件将promise作为prop传递给子组件，子组件通过resolve传递信息给父组件。

![图六](https://camo.githubusercontent.com/042737a3d3321825287edda26a28c0a58ce0b159/68747470733a2f2f706963342e7a68696d672e636f6d2f76322d33363937626563313430663931656666323463356132656330633031386231655f722e6a7067)


** （3）兄弟通信（两个组件有共同的 Parent）**

如图七

- 通过父组件做一个交换。比如说父组件造两个函数，分别传递给两个子组件，然后通过父组件作为一个桥接；

![图七]()

** （4）非父子、兄弟组件的任意两个组件通信**

如图八

> - 全局变量, 然后通过emmiter来通知；
> - context, 其实和全局变量没太大差别。
> - 这个实现需要借助第三方，redux等等。

![图八](https://camo.githubusercontent.com/b3a733d4f4af7b53a453fe6a2b2585ed627fbccb/68747470733a2f2f706963322e7a68696d672e636f6d2f76322d38333165636139313962626665616166333364366637323032386432613430615f722e6a7067)

## 八、总结

> - React 一切都是组件。组件可以不画东西，可以只负责通讯，可以只是参加一个副作用
> - 尽量使用 props，少用 state
> - React 哲学是单向数据流的哲学


## 参考链接

- [JSX 在线转义](https://babeljs.io/repl/#?babili=false&browsers=&build=&builtIns=false&spec=false&loose=false&code_lz=DwEwlgbgfAUABAuwBGBXALug9gOzugTwAcBTAXgCIBnVZAWzHQtkTgEgBlAQwhJjeAB6NJlywh4aEA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&sourceType=module&lineWrap=false&presets=react&prettier=false&targets=&version=6.26.0&envVersion=)

- [知乎Live：帮助你深入理解 React](https://github.com/Bian2017/Blog/issues/16)








