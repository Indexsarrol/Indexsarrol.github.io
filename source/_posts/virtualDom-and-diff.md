---
title: Vue源码之虚拟dom和diff算法（上）
author: Indexsarrol
date: 2022-04-15
categories: 
- Vue
tags:
- Vue
- Diff算法
---

## 前言

随着前端框架的发展，现在只停留会使用这些框架上是远远不够的，可以这么说，连面试可能都没法通过，所以现在的面试就要求我们不仅仅要会使用，而且要熟知其原理。虽然在工作当中，我们可能不会去要求我们去手写一些底层原理，但是懂了其原理，我们就可以快速的去定位问题，从而提升我们的工作效率，同时也能锻炼自己的编程能力。虚拟DOM和Diff算法这两个东西现在基本上每个前端同学都听过，而且或多或少的看过一些文章。但是大部分都处于一知半解的状态。懂，但又不是完全懂了。所以我最近也是有点苦恼，然后找了相关的资料狠狠的补习了一下。在这里也当作我的一个笔记吧。

## 什么是虚拟DOM和Diff算法

### 虚拟DOM是什么？

虚拟DOM，乍听感觉这个词非常牛逼，非常高大上。我们可以把词给拆开来看，“虚拟”和“`DOM`”，`DOM`这个词我们肯定不会陌生，他就是文本对象模型，我们不必再去赘述。那么怎么去理解“虚拟”这个词呢，假的！是“虚拟”可以理解为假的，不真实的，那么我们就可以将虚拟DOM这个词理解为不是真实的DOM——利用js对象来表示真实DOM。也就相当于说虚拟DOM就是真实DOM节点的一个js对象的映射。我们可以参考以下的图片:

![image-20220415133627012](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220415133627012.png)

我们从上图可以得知，真实DOM的节点都会对应一个虚拟DOM的对象。从而得出结论：

> 虚拟DOM就是真实DOM的js对象的一个描述。

### 为什么我们要用虚拟DOM？

在`jquery`时代，我们大家都知道，我们是通过操作真实DOM去处理页面的节点进行增删改的操作，可能对于我们来说就是一瞬间的事，但是对于计算机来说，可能需要进行成千上万次的操作，无疑这样是非常浪费性能的，这时候我们就引进了虚拟DOM，将需要更新的节点通过Diff算法进行最小化的更新，从而不再去操作真实DOM，仅对需要变化的节点进行更新操作，大大的节省的性能。

### Diff算法是什么以及为什么要使用它？

接下来，我们再聊聊Diff算法，那么什么是Diff算法呢。我们这里先直接给Diff算法下个定义：

> 简单来说Diff算法就是在虚拟DOM树从上至下进行同层比对，找出不同的地方进行最小量化的更新，从而间接更新真实DOM

我们怎么去理解Diff算法呢？这里举一个很简单的例子——装修。加入我现在有一个装修好的房子，现在呢，我想把主卧的床和客厅的沙发给换掉，那我们是不是直接把原来的沙发和床直接丢弃，然后弄个新的就可以了。那么Diff算法就是这么一个逻辑，

它会先去全局扫描一下，比较我变更前的床和沙发是否一样，如果不一样就直接将新的沙发和床把原来的给替换掉。如果一样的地方，比如电视机，没有变化，那就原封不动。从而节省的很多人力。放到程序中看，如下图：

![image-20220415135418566](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220415135418566.png)

在这里我们可以看到，我们在新的DOM元素上添加了一个新的`<span>`标签以及`<li>`标签，那么Diff算法就可以进行精细化的比较，从而实现最小化的更新。

## snabbdom简介以及测试环境搭建

### 什么是snabbdom？

`snabbdom`是瑞典语的一个单词，意为“速度”，而我们熟知的`Vue.js`，其源码也借鉴了`snabbdom`，可以说`snabbdom`是虚拟DOM的鼻祖。

### 测试环境搭建

想要知道其内部怎么实现的，我们就需要自己手动的去搭建一个测试环境来进行代码的调试，以及后面手写代码。首先创建文件夹，命名为`study-snabbdom`，然后执`行npm init -y`

#### 安装

`snabbdom`有很多版本，为了方便，我们就以2.1.0的版本作为测试环境中的版本：

```bash
npm install snabbdom --save
```

### 搭建webpack

这里我就不在多赘述了，想了解的可直接前往`webpack`[官网](https://webpackjs.com/)进行文档查阅。在这里仅把配置项放出来。

```javascript
// webpack.config.js
module.exports = {
  entry: './src/index.js',
  output: {
    publicPath: 'dist',
    filename: 'bundle.js'
  },
  devtool: 'cheap-module-source-map', 
  devServer: {
    port: 8080,
    contentBase: 'www'
  }
}
```



## h函数

### h函数是什么？

什么是`h`函数呢，简言之：

> 就是用来生成虚拟节点（`vnode`）的一个方法。

### h函数如何使用？

比如我们可以这么去调用`h`函数:

```javascript
h('a', { props: { href: 'https://www.baidu.com' } }, '百度');
```

这样我们就能得到这样的一个虚拟节点：

```
{
	sel: 'a', // 选择器
	data: { // 相关属性
		props: {
			href: 'https://www.baidu.com'
		}
	},
	text: '百度', // 标签内文本内容
}
```

那么在页面上，他就表示一个真实的A标签的节点

```html
<a href="https://www.baidu.com">百度</a>
```

我们通过上面可以知道，`h`函数是负责生成一个虚拟节点的方法，那么，虚拟节点里面都有哪些属性呢？

```json
{
	"sel": "div", // 选择器
	"data": {}, // 属性样式类名等
    "text": '12345', // 文本内容
    "children": undefined, // 子节点
    "elm": undefined, // 真实DOM节点
    "key": undefined // 唯一值
}
```

### 演示h函数使用以及上树

在刚刚搭建的环境中，我们在`index.js`文件中，导入`snabbdom`相应的模块：

```js
import { init, h } from 'snabbdom';

const vnode = h('a', { props: { href: 'https://www.baidu.com' } }, '百度');

console.log(vnode);
```

这是我们可以看到控制台上面输出了一个js对象：

![image-20220415144500575](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220415144500575.png)

那么虚拟节点有了，我们怎么去将这个节点渲染到页面上面呢？这时候我们就要调用`snabbdom`中的`patch`函数，来帮助我们完成上树操作：

```js
import { init, propsModule, h } from 'snabbdom';
const patch = init([propsModule]);
const vnode = h('a', { props: { href: 'https://www.baidu.com' } }, '百度');
const container = ducument.getElementById('container');

patch(container, vnode);

```

我们打开页面就可以看到如下：

<img src="https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220415145047553.png" alt="image-20220415145047553" style="zoom: 67%;" />

这样我们就相当于上树了，并且`href`属性内的链接是可以点击的。

### h函数更多写法

在上面我们说过`h`函数的一些简单的用法，这里我们要介绍一下`h`函数的新用法——嵌套，话不多说，直接上代码:

```js
h('ul', {}, [
	h('li', {}, '牛奶'),
	h('li', {}, '咖啡'),
	h('li', {}, '可乐')
])
```

那么以上代码，转换成虚拟DOM就变成这样:

```js
{
	sel: 'ul',
	data: {},
	children: [
		{ set: 'li', text: '牛奶' },
		{ set: 'li', text: '咖啡' },
		{ set: 'li', text: '可乐' }
	]
}
```

我们的`h`函数，其内部实现了函数的重载，所以参数传递的方式多种多样，例如以下的写法都是可以的：

```js
h('div') // ===> { sel: 'div', text: undefined, children: undefined }
h('div', '我是div');  // ===>  { sel: 'div', text: '我是div', children: undefined }
h('div', []);  // ===>  { sel: 'div', text: undefined, children: [] }
h('div', h());  // ===>  { sel: 'div', text: undefined, children: [{ sel: undefind, text: undefined, children: undefined }] }
h('div', h('div', '我是内部的盒子')); // ===>  { sel: 'div', text: undefined, children: [{ sel: 'div', text: '我是内部的盒子', children: undefined }]
h('div', {}, '12354') // ===>  { sel: 'div', data: {}, text: '12354', children: undefined }
```

足以看出，h函数的使用是非常的灵活！

## 如何手写一个简单版的h()函数

由于`snabbdom`源码内的h函数，实现了函数重载，为了方便，这里我们统一采用传入三个参数的方式进行实现。那我们就开始吧！我们先观察一下这个方法的返回值都是一个对象的形式，而且固定，所以我们可以把返回的对象封装为一个函数`vnode`：

### vnode函数

`vnode`函数传入选择器(sel)、数据项(data)、文本(text)、子节点(children)，真实DOM节点(elm)返回一个新的虚拟DOM对象，新建一个`vnode.js`文件：

```js
// vnode.js
export default function vnode(sel, data, children, text, elm) {
    const key = data.key ? data.key : undefined; // 查询数据项中是否有key
    return {
        sel,
        data,
        children,
        text,
        elm,
        key
    }
}
```

### h函数

首先，新建一个名为`h.js`的函数，并且导入`vnode.js`，我们在一开始也说过，只实现三个参数的方法。分别是以下三种：

```
h('div', {}, '文本');
h('div', {}, []);
h('div', {}, h('div'));
```

代码如下：

```js
import vnode from './vnode.js';

// 判断是否为文本类型
function isText(value) {
    return typeof value === 'string ' || typeof value === 'number';
}

// 判断是否为子节点数组类型
function array(value) {
    return Array.isArray(value);
}

// 判断是否为对象且包含sel属性
function isObjectAndExitSel(value) {
    return typeof value === 'object' && value.sel
}

export default function h(sel, data, c) {
    // 1.判断当前传入参数是否为3个，如果不是直接报错
    if (arguments.length !== 3) {
        throw new Error('参数个数必须为3个!')
    }
    // 2.判断c参数是文本还是children
    if (isText(c)) {
        // 2.1如果是文本类型的节点，则直接返回vnode函数
        return vnode(sel, data, undefined, c, undefined);
    } else if (array(c)) {
        // 2.2 如果c是数组
        for (let i = 0; i < c.length; i++) {
            const children = [];
            if (!isObjectAndExitSel(c[i])) {
                throw new Error('传入的数组有一项不是h函数')
            }
            children.push(c[i]);
        }
        return vnode(sel, data, children, undefined, undefined);
    } else if (isObjectAndExitSel(c)) {
        // 2.3 如果是h函数，则被认为是单纯的对象
        const children = [c];
        return vnode(sel, data, children, undefined, undefined);
    } else {
        // 以上条件全部不满足
        throw new Error('第三个参数类型不满足要求')
    }
}
```

